# Build image
````sh
docker build -t ssia-ch3-ex1:1.0 .
````

# Run image
````sh
docker run -p 8080:8080 -t ssia-ch3-ex1:1.0
````

# Test (happy path)
````shell
curl -u john:12345 http://localhost:8080/hello
````

# Test avec un mauvais user/mdp, pas de retour...
````shell
curl -u ggn:12345 http://localhost:8080/hello
````

# Actuator
````shell
curl http://localhost:8080/actuator
````
````shell
curl http://localhost:8080/actuator/health
````

## Arrêter le conteneur
````sh
docker ps
````
````sh
docker stop $(docker ps -q --filter ancestor=ssia-ch3-ex1:1.0)
````