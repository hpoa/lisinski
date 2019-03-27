Running a node ( without sealing )

```
docker run -v `pwd`/lisinski.json:/home/parity/lisinski.json nodefactory/parity-goerli:latest --chain /home/parity/lisinski.json --no-warp
```
