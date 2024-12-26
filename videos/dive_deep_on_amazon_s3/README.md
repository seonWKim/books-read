# [AWS re:Invent 2023 - Dive deep on Amazon S3 (STG314)](https://www.youtube.com/watch?v=sYDJYqvNeXU&list=WL&index=3)

- Reactive -> Threat modeling -> Proactive 

## Threat Modeling 
 
- Write a lot of docs 
- Threat reviews 
  - Actual written document 
  - Interactive working meeting 
  - Review with experienced team members 
  - Learning opportunity for junior engineers 

## What is S3 

![overview.png](overview.png)

### S3 front end is big 

- 1PB/sec 
- The treat of scale and going wide 
  - Mechanisms for mitigation 
    - Use multipart upload to parallelize puts 
      - ![multipart_upload.png](multipart_upload.png)
      - ![multipart_upload_2.png](multipart_upload_2.png)
    - Use ranges to parallelize gets 
      - ![multipart_get.png](multipart_get.png)
    - Spread requests across many IPs in the fleet 
      - ![multipart_scaling.png](multipart_scaling.png)

### S3 Index is big 

- 350 trillion objects 
  - it's constantly growing  

![index_partitioning.png](index_partitioning.png)
- But distribution differs by characters -> hotspot occurs 

![index_hotspot_scaling.png](index_hotspot_scaling.png)
- S3 make use of prefix(any string of characters after the bucket name) to partition data 

![prefix_limitation.png](prefix_limitation.png)
- If exceeded, 503 is returned

![prefix_scaling.png](prefix_scaling.png)
- You can split the prefix the scale 

![bad_key_naming.png](bad_key_naming.png)
- Bad key naming strategy as day2 can't leverage split on day1

![better_key_naming.png](better_key_naming.png)
=> better key naming 

### Storage 

- Millions of hard drives 
- Achieving 11 9s of durability
  - end-to-end integrity checking of requests
    - ![integrity_check.png](integrity_check.png)
      - checksums 
    - ![redundant_storage.png](redundant_storage.png)
      - erasure encoding 
      - monitors for corruption, replicate when detecting corruption
  - ![redundant_storage_2.png](redundant_storage_2.png)
    - data always stored on redundant devices  
  - periodic durability auditing for data at rest

### AZ loss 

- Multi-AZ by default 
