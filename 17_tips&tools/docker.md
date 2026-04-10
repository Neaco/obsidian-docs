
##### steps : Write dockerfile > build > run / tag + publish
docker build -t image-name .
docker run --rm -it -p hostport:containerport --user 33 imagename
docker tag img ghcr.io/username/imagename:latest
echo github_pat | docker login ghcr.io -u neaco --password-stdin 
docker push ghcr.io/username/imagename:latest
##### Run it with bash instead of default : 
docker run ... --entrypoint /bin/sh

##### Exec inside container : 
docker exec -it containername /bin/sh