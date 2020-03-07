---
layout: post
title: MongoDB Golang JSON (Mongo Alternatives?)
---
I need a database, since working with a JSON file in the filesystem was not going far enough.  Need to query objects, delete objects, sort, etc.  
Also I like the idea of using Golang to deal with JSON at the client-server interface, since it's really easy in Golang.  Then Golang can talk to the database.  
Initial googling suggested either postgres or Mongo, and I'm trying Mongo because I haven't yet.  
  
These instructions worked: `https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-16-04`
  
Nice tutorial here.  `https://medium.com/@maumribeiro/a-fullstack-epic-part-i-a-rest-api-in-go-accessing-mongo-db-608b46e969cd`
  
# Main Concepts
Basically what happens is there's a POST request, and the body contains JSON.  The JSON is unmarshalled into a golang struct.  The struct is passed to `c.Repository.AddAlbum`.  
Repository is an empty struct with methods, eg. GetAlbums() , AddAlbum().  There are constants in the author's code but I imagine you could put the constants eg. SERVER,DBNAME,DOCNAME in the Repository struct, so there could be 1 Repository per user.  
The methods (eg. GetAlbums) will use the golang Mongo driver, mgo.  The database connection object has methods so we can do crazy syntax like this:  
```
    c := session.DB(DBNAME).C(DOCNAME)
    results := Albums{}
    if err := c.Find(nil).All(&results); err != nil {
        fmt.Println("Failed to write results:", err)
    }
```
Let's look at AddAlbum:  
```
func (r Repository) AddAlbum(album Album) bool {
    session, err := mgo.Dial(SERVER)
    defer session.Close()
    
    album.ID = bson.NewObjectId()
    session.DB(DBNAME).C(DOCNAME).Insert(album)

    if err != nil {
        log.Fatal(err)
        return false
    }
    return true
}
```
An Album is passed to AddAlbum.  The ID field is generated here and set.  Then, the Album is Inserted into the database.  Cool!  
UpdateAlbum works similarly except one line is different:  
`    session.DB(DBNAME).C(DOCNAME).UpdateId(album.ID, album)` .  It finds the album with the ID, then replaces it.  
I see now, basically what the database is doing is attaching an ID to each JSON object.  This is a critical missing piece, when handling JSON naively.
  
# OK let's install it on the server
Well just for fun let's do it with a docker container.  
`sudo docker run -d -p 27017:27017 -v ~/data:/data/db mongo` easy.  
Crap, 366MB image.  Another 100MB to install the client tools.  Maybe I can get away with only using the golang libraries?  
Cool, this minimal snippet worked!
```
func main() {
  fmt.Println("Hello1")
  session, err := mgo.Dial("localhost")
  if err != nil {
    fmt.Println("Hello2")
    panic(err)
  }
  defer session.Close()
  fmt.Println("Hello3")
}
```

# Now, the Internet says check security
And yes, the MongoDB server is accessible from the public.  
Let's use a docker-compose.yml to put in our own .conf.  
Oh, weird, turns out the original installation had already set bindaddress to 127.0.0.1.  So... even though the Mongo server only accepts connections from 127.0.0.1, because it's in a Docker container, it is still reachable from the outside world?  Does Docker change the source IP in the port forward?  Does that mean this same issue will happen for any server inside a container?  
  
OK my solution now is to try `-p 127.0.0.1:27017:27017` .  Yes!! That is the solution.  Now I am unable to access from outside.  And I am still able to access from the simple golang snippet above.  
  
# Hmm, weirdness getting a Flask container to talk to a Mongo container.
I'm using Flask as my API server.  It is in a container.  It receives requests and then executes shell scripts.  The shell scripts are located in a host directory that is volume mounted to the container.  I compiled the exact golang dummy program above and put it in that directory.  It executes fine on the host, but when executing in the container, the Mongo server cannot be found.  
At first I thought, there is no port forwarding that gives the Flask container access to localhost:27017 on the host.  The Flask container did have 9002:5000, which makes the Flask default port 5000 accessible by the public at port 9002.  So I tried to do a 27017:27017 in the Flask container.  But no!  Docker could not bind to 27017, because Mongo was already bound to it.  
So, I still don't understand this but my solution was to change the Flask container to use Host Networking, ie. `network_mode:host`
# Oh no... that ended up breaking Flask!
Turns out, setting `network_mode:host` overrides the port forward I had `9002:5000`, since the Flask server stopped responding to 9002, and started responding to 5000!
# OK I will just configure the Flask port.  Using network_mode:host.
Ugh this sucks.  I couldn't figure out how to block public access to Mongo.  So I actually put Mongo in the docker network, doing a port publish `22222:21017`.  The Flask is using `network_mode:host`.  I did verify Flask can access Mongo, and the public cannot.
  
# I'm reading stupid hate comments about MongoDB... why?
`https://getstream.io/blog/building-a-performant-api-using-go-and-cassandra/`
I looked at cassandra in this article and wow... I have no desire to use it after looking at the code examples.  

