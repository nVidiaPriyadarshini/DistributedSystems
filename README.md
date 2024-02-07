Resource material from course: [https://www.udemy.com/course/building-modern-distributed-systems-with-java/learn/lecture/29391314#overview]

![Problem Statement](https://github.com/nVidiaPriyadarshini/DistributedSystems/blob/main/Remote%20Procedure%20Call.png)

Steps to introduce Cluster Coordination in Tinyurl application
1. Start gateway java process
2. Start two backend instances
   In backend logs observe details of elected leader and tinyurl service id details
   - Leader elected [sequence-leader]: b9c4da2b-63aa-4f20-9e80-2820b1ee981e
   - Registering service with consul: NewService{id='tiny-url-backend-b9c4da2b-63aa-4f20-9e80-2820b1ee981e', name='tiny-url-backend', tags=[],
3. Start docker using below command
   - docker-compose -f ./docker-compose.yml up
4. Log into one of the etcd containers
   - docker exec -it etcd-2 /bin/sh
5. Use etcd get command to obtain key information
   - etcdctl --endpoints etcd-1:21379 get --prefix "sequence-leader/"
>> sequence-leader/78098d84222efc04
b9c4da2b-63aa-4f20-9e80-2820b1ee981e
sequence-leader/78098d84222efc09
309b5d0a-e0ee-4363-92dc-67766b276c2a

6. Get details of key 1
   - etcdctl --endpoints etcd-1:21379 get sequence-leader/78098d84222efc04 --write-out=json
>> {"header":{"cluster_id":18396408571102523720,"member_id":11047135559872888732,"revision":2,"raft_term":12},"kvs":[{"key":"c2VxdWVuY2UtbGVhZGVyLzc4MDk4ZDg0MjIyZWZjMDQ=","create_revision":2,"mod_revision":2,"version":1,"value":"YjljNGRhMmItNjNhYS00ZjIwLTllODAtMjgyMGIxZWU5ODFl","lease":8649600157990452228}],"count":1}
7. Get details of key 2

   - etcdctl --endpoints etcd-1:21379 get sequence-leader/78098d84222efc09 --write-out=json
>> {"header":{"cluster_id":18396408571102523720,"member_id":11047135559872888732,"revision":3,"raft_term":12},"kvs":[{"key":"c2VxdWVuY2UtbGVhZGVyLzc4MDk4ZDg0MjIyZWZjMDk=","create_revision":3,"mod_revision":3,"version":1,"value":"MzA5YjVkMGEtZTBlZS00MzYzLTkyZGMtNjc3NjZiMjc2YzJh","lease":8649600157990452233}],"count":1}
8. Get details of lease of key 1
   - etcdctl --endpoints etcd-1:21379 lease timetolive --keys 78098d84222efc04
>> lease 78098d84222efc04 granted with TTL(10s), remaining(9s), attached keys([sequence-leader/78098d84222efc04])
9. Invoke curl command to issue a url shorten process.
   - curl -d "http://www.war.com" -H "Content-type: application/text" -X POST http://localhost:8080/short-url
>> http://localhost:8080/url/0%   
>> http://localhost:8080/url/5%
>> http://localhost:8080/url/1%
>> http://localhost:8080/url/6%
> 
> ids: 0-5 and 5-10 obtained
10. Review state of counter key
    - etcdctl --endpoints etcd-1:21379 get --prefix "tiny-url/counter"
>> 10