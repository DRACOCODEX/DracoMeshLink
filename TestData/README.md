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
Example Output: `21924		856210`, `{"data":{"height":857354}}`

+ Snapshots
```console
   docker exec miner miner snapshot list
```
Output:

  Height 856081 Hash <<34,51,3,147,80,48,2,168,132,90,170,17,53,50,104,236,208,172,17,223,246, 38,105,223,217,41,56,133,41,232,120,234>> (<<"22330393503002a8845aaa11353268ecd0ac11dff62669dfd929388529e878ea">>) Have true

  Height 855361 Hash <<188,219,164,245,122,49,19,6,143,51,228,135,195,157,222,250,213,160,84, 229,237,17,27,226,176,34,136,32,30,248,79,97>> (<<"bcdba4f57a3113068f33e487c39ddefad5a054e5ed111be2b02288201ef84f61">>) Have true

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
  + Check that Miner is Connect to Packet forwarder
  ```console
     tail -F /home/pi/miner_data/log/console.log | grep lora
  ```
  Output Data: `2021-05-24 05:03:09.166 1 [info] <0.1345.0>@miner_lora:handle_udp_packet:390 PULL_DATA from 12273815315514654720 on 44948`

+ Back Up Swarm Keys
```console
   scp pi@IP-ADDRESS:/home/pi/miner_data/miner/swarm_key .
```
