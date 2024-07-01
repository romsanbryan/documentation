# Maven
## Versioning
### Crear versioes snap
    mvn --batch-mode release:update-versions -DdevelopmentVersion=<version>-SNAPSHOT
### Crea versiones release
    mvn versions:set -DremoveSnapshot
