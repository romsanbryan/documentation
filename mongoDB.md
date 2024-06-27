# MongoDB
## BBDD
### ver BBDD
    show dbs
### ver colecciones
    show collections
### Utilizar/Crear BBDD
Si se crea, no se vera creada fisicamente hasta que se le asigna su primera coleccion
    use <name>
#### Borrar BBDD
    db.dropDatabase(<name>)
## Informacion
### Scripts
Se pueden crear script que realicen funcionalidades. Estos scripts van en JS, para ejecutarlos:

    - Windows: 
        start mongosh.exe <script.js>
### Import colecciones
 -     mongoimport --uri mongodb+srv://<user>:<pass>@<url>/ <file.json> -d <BBDD> -c <collection> --jsonArray
 -     mongoimport --uri mongodb+srv://<user>:<pass>@<url>/ <file.json> -d <BBDD> -c <collection>
### Tipo de datos
- String
- Integer / Double 
- Boolean
- Date (ISO 8601)
- Array
- Objects
- Datos Binarios (imagenes, documentos, pdf, archivos de audio/video, etc)
- ObjectId (id del documento, por defecto se crea solo)
- null
## Comandos
### Insertar
-     db.<collection>.insertOne(<json>)
-     db.<collection>.insertMany([<json>,<json> ]) 
#### Ej:
    db.coches.insertOne({"matricula":"BRYAN","marca":"Seat","version":"confort","kms":90000,"fechaMatriculacion":"20230221T000000"}) 
### Borrar
-     db.<collection>.deleteOne(<json>)
-     db.<collection>.deleteMany([<json>,<json> ]) 
-     db.<collection>.deleteMany({})
#### Ej:
    db.coches.deleteOne({"estado":true}) 
### Actualizar
-     db.<collection>.updateOne(<filter>, {$set: {json} })
-     db.<collection>.updateMany(<filter>, {$set: {json} })
#### Ej:
    db.coches.updateOne ({"marca": "SUV"}, {$set {"precio": 30000} })
    db.coches.updateOne ({"marca": "SUV"}, {$set {"precio": 30000}, $currentDate: {lastModified: true} })
    db.coches.updateMany({"marca": "moto"}, { $push : {"extras": "casco"} })
Con $set se modifican los campos existentes
con $push se añaden en un array
#### Replace
Adicionalmente existe el replace para modificar el documento por uno completamente nuevo. Se mofica al COMPLETO
-     db.<collection>.replaceOne(<filter>, <update>) 
### Consultas
-     db.<collection>.find()
-     db.<collection>.find(<filter>)
#### Adicionales
##### Formateo
    db.<collection>.find().pretty()
##### Contar
    db.<collection>.find(<filter>).count()
    db.<collection>.countDocument(<filter>)
    db.<collection>.count()
##### Limitar
    db.<collection>.find().limit(<number>)
#### Ej.
##### Devuelve los que tienene matematicas y fisica en este mismo orden
    db.profesores.find({[especialidad:["matematicas", "fisica"]]}) 
#### Devuelve los que tienene matematicas y fisica da igual el orden
    db.profesores.find({[especialidad: {$all: ["matematicas", "fisica"]}]}) ->  solo 
#### Devuelve los profesores que tienen quimica
    db.profesores.find({especialidad:"quimica"}) 
#### Devuelve si el id dentro de asignatura es el A0002
    db.profesores.find({"asignatura.id":"A0002"})
#### Devuvelve las asignaturas que los creditos sean menor a 6
    db.profesiores.find({"asignatura.creditos": {$lt 6}}) 
#### Devuelve los que tienen mas de 5 años registrados y son menores de 20 años
    db.contacts.find({"registered.age": {$gte: 5}, "dob.age": {$lte: 20} })
#### Devuelve los que tienen mas de 5 años registrados O (se hace con el operador "$or" ) edad menor a 20
    db.contacts.find({$or: [{"registered.age": {$gte: 5}}, {"dob.age": {$lte: 20}}] })
## Indices
    db.<collection>.getIndexes()
    db.<collection>.createIndex(<documento index>) 
    db.<collection>.dropIndex(<id del indice>)
    db.<collection>.dropIndexes()
### Adicional
#### Crear indice a partir de una expresion
    db.<collection>.createIndex({"dob.age": 1}, {partialFilterExpression: {"dob.age": {$gt: 65} } }) 
#### Indice con TT, para controlar documentos por fechas
    db.<collection>.createIndex(<documento index>, <expiracion>) 
### Ej.
#### Mostrar indices
    db.profesores.getIndexes() -> muestra indices
#### Crear indice en orden ascendente (matricula) y descendente (edad)
    db.collection.createIndex({matricula: 1, edad: -1})
#### Borrar indice
    db.collection.dropIndex('edad_-1') 
#### Borrar todos los indices
    db.collection.dropIndexes()
#### Indices unicos
Para evitar la duplicidad de valores
    db.collection.createIndex({nombre: 1}, {unique: true})
#### Crear indice a traves de una subconsulta
    db.contacts.createIndex({"dob.age": 1}, {partialFilterExpression: {"dob.age": {$gt: 65} } })
#### TT de expiracion de 10 segundos
    db.users.createIndex({createAt: 1}, {expireAfterSeconds: 10})
## Informacion
### Estadisticas de consultas
    db.<collection>.explain("executionStats").find(<filter>) 
#### Ej
##### Ver estadisticas de una consulta
    db.contacts.explain("executionStats").find({"registered.age": {$gt: 11}}) 
## Aggregate
[Info](https://www.mongodb.com/docs/manual/reference/operator/aggregation-pipeline/)
### $match
Similar a find

    db.movies.aggregate([{$match: {year: {$gte: 2000}} }])
### $group
Agrupaciones

    db.movies.aggregate([{$match: {year: {$gte: 2000}}}, {$group: { _id: "$year", total: {$sum: 1} }}])

            [
                { _id: 2007, total: 872 },
                { _id: 2010, total: 970 }
            ]
### $sort
Ordenar. Ademas de ordenar los campos que existan, si se le hace una agrupacion se puede ordenar por esos campos tambien

    db.movies.aggregate([{$match : { "imdb.rating": {$gt:9}}}, {$sort: {year: -1}} ])
### $project
Filtra datos que queremos mostrar, con 0 eliminas y con 1 muestras

    db.movies.aggregate([{$project: {_id: 0, title: 1, year: 1}}]
Ademas puedes personalizar campos para que utilice los valores de los documentos con el $

    db.movies.aggregate([{$project: {title: 1, year: 1, rating : "$imdb.rating"}}])
Se puede utilizar con concat para concatenar valores

    db.movies.aggregate([{$project: {name: { $concat: [{$toUpper: "$title"}, "-", {$toString: "$year"}] }, rating : "$imdb.rating"}}])
### $skip
Para hacer paginados (muestra los 10 pero se salta los 10 primeros)

    db.movies.aggregate([{$project: {_id: 0, title: 1, year: 1}}, {$sort: {year: 1}},{$skip: 10}, {$limit: 10} ])
### $out
Hace un filtro y guarda en una nueva coleccion

    db.movies.aggregate({$project: {_id: 0, title: 1, wins: "$awards.wins"}}, {$sort: {wins: -1}}, {$limit: 10}, {$out: "masPremiadas"}  ) 
### $face
Permite hacer subconsultas

    db.movies.aggregate([{ $match: { "cast": "Lisa Kudrow" } }, { $facet: { "rating": [{ $project: { _id: 0, title: 1, rating: "$imdb.rating" } }, { $sort: { rating: 1 } }] } }] )