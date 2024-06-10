1. mkdir count-messages && cd count-messages

2. docker compose up -d
 
3.  docker exec -it ksqldb bash -c 'echo -e "\n\n  Waiting for startup… \n"; while : ; do curl_status=$(curl -s -o /dev/null -w %{http_code} http://ksqldb:8088/info) ; echo -e $(date) " ksqlDB server listener HTTP state: " $curl_status " (waiting for 200)" ; if [ $curl_status -eq 200 ] ; then  break ; fi ; sleep 5 ; done'


4. docker exec kcat \
    kcat -b broker:29092 -C -t pageviews -e -q | \
    wc -l

5. docker compose down
    
