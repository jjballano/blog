# Docker compose
#####(01/03/2017)

I wrote a post about how to [add postgres to my pet project with docker](postgres_docker.md) but I wasn't happy with the soluction because make nonsense to run a few containers that you could potencially need, one by one. That's why I started to use [docker-compose](https://docs.docker.com/compose/). 
The idea is just running one thing and have all your environment ready to use and docker-compose make this very easy.

I created a file called docker-compose.yml in the root of my project and added the following configuration:

```
version: "2"
services:
  db:
    image: "postgres:9.6.2"
    ports:
      - "5432:5432"
    volumes:
      - data:/var/lib/postgresql/data  

volumes:
  data:
```

As you can see, we have a db service with the postgres configuration, very similar to the 'docker run' with had previously.

So now, we need to execute the following in order to have your database up & running:

```
docker-compose up
```

Easy, right? But we have done nothing special, we changed from a longer run sentence to an easier one to remember, nothing else. What if we want to run everything in just one step? 

My application is a rails one, so first of all we need to add a file called 'Dockerfile' to the root of our project in order to tell Docker how to build the container of the application.

```
FROM ruby:2.3.3
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /time
WORKDIR /time
ADD . /time
RUN bundle install
```

I think it is quite clear what's doing. 
Start from the image of ruby 2.3.3, installing the dependencies a rails project requires with apt-get and then getting the folder of the project ready in the container. 

And now, our docker compose file is something like this:

```
version: "2"
services:
  db:
    image: "postgres:9.6.2"
    ports:
      - "5432:5432"
    volumes:
      - data:/var/lib/postgresql/data  
  web:
    build: .
    command: /myapp/bin/rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db

volumes:
  data:
```

As you can see, our web service will run the rails in the same way you do it manually. The configuration "depends_on" tells Docker to wait until another service is run before start running the current one.

So now, we should execute:

```
docker-compose build
```

This will create the 'web' container depending on Dockerfile created before. 

Now we can get all system up & running with just one sentence:

```
docker-compose up
```

And you can access your application at http://192.168.99.100:3000. As I'm using a Mac (same for Windows) and need a VM created with docker-machine, my container IP is 192.168.99.100 (you can get the IP of your with docker-machine ip VM_NAME). If you are in a Linux environment, I guess host would be localhost.

You can execute any command within the container with docker-compose, i.e.:
```
docker-compose run web rails db:migrate
docker-compose run web rails c
etc...
```

One last configuration stuff. Docker has created a host name for your database called 'db' (same name as service), so in your application you should use this host name in postgres configuration.

```
DATABASE_URL=postgres://postgres:@db:5432/MY_DB
```