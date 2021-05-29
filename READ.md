docker run -it -p4040:4040 -p8089:8089 -v /Users/jackie/Desktop/CS144:/home/cs144/shared --name locust junghoo/locust

# Inside another terminal not in container and just local 
docker start tomcat
docker start mean

# We only need to start one server since proj3 was merged w proj4
docker exec -d mean npm start --prefix /home/cs144/shared/project3/blog-server
docker network create cs144-net

docker network connect cs144-net locust
docker network connect cs144-net tomcat
docker network connect cs144-net mean

# In another host which will run locust container
docker exec --user cs144 -it locust /bin/bash

locust -f read_tomcat.py --host=http://tomcat:8888 --headless -u 10 -r 10 -t 30s
locust -f write_tomcat.py --host=http://tomcat:8888 --headless -u 10 -r 10 -t 30s

locust -f read_node.py --host=http://mean:3000 --headless -u 10 -r 10 -t 30s
locust -f write_node.py --host=http://mean:3000 --headless -u 10 -r 10 -t 30s

curl http://mean:3000

curl --request POST --header "Content-Type: application/json" --data '{"username": "cs144", "postid": 2, "title": "Hello", "body": "***World***"}' --cookie "jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjIzMjY0NzEsInVzciI6ImNzMTQ0IiwiaWF0IjoxNjIyMzE5MjcxfQ.6GS4rqHWvuj-VOXWoTRtz93L9OYVikJbRuTMJshob8w" http://mean:3000/api/posts

locust -f mixed_tomcat.py --host=http://tomcat:8888 --headless --reset-stats -u 300 -r 100 -t 30s --only-summary |& tee summary_tomcat.txt

Response time percentiles (approximated)
 Type     Name                                                              50%    66%    75%    80%    90%    95%    98%    99%  99.9% 99.99%   100% # reqs
--------|------------------------------------------------------------|---------|------|------|------|------|------|------|------|------|------|------|------|
 GET      /editor/post?action=open                                          230    270    290    310    380    560    940   1100   1800   2600   2600   6041
 POST     /editor/post?action=save                                          250    290    310    330    440    640    990   1200   1700   1900   1900   1485
--------|------------------------------------------------------------|---------|------|------|------|------|------|------|------|------|------|------|------|
 None     Aggregated                                                        230    270    300    320    390    580    950   1200   1800   2600   2600   7526

locust -f mixed_node.py --host=http://mean:3000 --headless --reset-stats -u 30 -r 100 -t 30s 
locust -f mixed_node.py --host=http://mean:3000 --headless --reset-stats -u 30 -r 100 -t 30s --only-summary |& tee summary_node.txt