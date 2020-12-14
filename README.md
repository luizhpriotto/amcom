<!-- ABOUT THE PROJECT -->
## Web Application using NODE.JS

### Important Points

* you must to start the prd em qas shark's services aplication and the Traefik using the YML. (docker stack deploy -c traefik.yml shark --with-registry-auth)
* In the pipeline, prd services on docker swarm always be updated and always branch master is used.
* Qas shark's services can be reused or created. You can access with a new dinamic DNS.
* When you are creating the acme.json, remeber the set the permission to 600 (chmod 600)

## Services
* [Jenkins](http://10.1.0.60:9090)
* [Traefik](http://10.1.0.60:9081)
* [Nexus](http://10.1.0.60:8081)
* [Rancher](http://10.1.0.60:8280)
* [Vizualizer](http://10.1.0.60:8282)
