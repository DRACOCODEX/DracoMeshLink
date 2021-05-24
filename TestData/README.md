# TESTING the MINER and Network

## Container-Miner
+ Verify container is running
```console
   docker ps
```
+ Verify connection to network
```console
  docker exec miner miner peer book -s
```
  Screenshot: https://github.com/Dragons-Den/HELIUM-MVP/blob/main/TestData/Network%20Connected.png

+ Verify Blockchain Processing
```console
  docker exec miner miner peer book -s
```
```console
   curl https://api.helium.io/v1/blocks/height
```
Example Output: `21924		856210`, {"data":{"height":857354}}

+ Useful Commands for Testing
  + Log files
  ```console
   docker logs miner
   ```
  ```console
     tail -F /home/pi/miner_data/log/console.log
  ```
  Output: LoRaWAN traffic and blockchain commits

  + List Docker images
  ```console
     docker images
  ```
