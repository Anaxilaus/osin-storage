# dockertest

[![Build Status](https://travis-ci.org/ory-am/dockertest.svg)](https://travis-ci.org/ory-am/dockertest)

Use docker to run your Go language (integration) tests against persistent services like **MySQL, Postgres or MongoDB** on **Microsoft Windows, Mac OSX and Linux**! Dockertest uses [docker-machine](https://docs.docker.com/machine/) (aka [Docker Toolbox](https://www.docker.com/toolbox)) to spin up images on Windows and Mac OSX as well!

A suite for testing with docker. Based on  [docker.go](https://github.com/camlistore/camlistore/blob/master/pkg/test/dockertest/docker.go) from [camlistore](https://github.com/camlistore/camlistore).
This fork detects automatically, if [docker-machine](https://docs.docker.com/machine/) is installed. If it is, you are able to use the docker integration on Windows and Mac OSX as well without any additional work. To avoid port collisions when using docker-machine, dockertest chooses a random port to bind the requested image to.

## Why should I use dockertest?

When developing applications, you most certainly encounter services talking to a database. (Unit) Testing these services can be quite a pain because mocking database/DBAL is horrible. Making slight changes to the schema implies rewriting at least some, if not all of the mocks. The same goes for API changes in the DBAL.  
To avoid this, it is smarter to test these specific services against a real database which is destroyed after testing. Docker is the perfect tool to solve this for you, as you can spin up containers in a few seconds. This library gives you easy to use commands for spinning up Docker containers and using them for your tests.

## Usage

The usage of dockertest is very simple. For now, MongoDB, Postgres and MySQL containers are supported out of the box. Feel free to extend this list by contributing.

### MongoDB Container

```go
import "github.com/ory-am/dockertest"
import "gopkg.in/mgo.v2"
import "time"

func Foobar() {
  // Start MongoDB Docker container. Wait 1 second for the image to load.
  containerID, ip, port, err := dockertest.SetupMongoContainer(time.Duration * 10)

  if err != nil {
    return err
  }

  // kill the container on deference
  defer containerID.KillRemove()

  url := fmt.Sprintf("%s:%d", ip, port)
  sess, err := mgo.Dial(url)
  if err != nil {
    return err
  }

  defer sess.Close()
  // ...
}
```

### MySQL Container

```go
import "github.com/ory-am/dockertest"
import "github.com/go-sql-driver/mysql"
import "database/sql"
import "time"

func Foobar() {
    // Wait 10 seconds for the image to load.
    c, ip, port, err := dockertest.SetupMySQLContainer(time.Second * 10)
    if err != nil {
        return
    }
    defer c.KillRemove()

    url := fmt.Sprintf("mysql://%s:%s@%s:%d/", dockertest.MySQLUsername, dockertest.MySQLPassword, ip, port)
    db, err := sql.Open("mysql", url)
    if err != nil {
        return
    }

    defer db.Close()
    // ...
}
```
### Postgres Container

```go
import "github.com/ory-am/dockertest"
import "github.com/lib/pq"
import "database/sql"
import "time"

func Foobar() {
    // Wait 10 seconds for the image to load.
    c, ip, port, err := dockertest.SetupPostgresContainer(time.Second * 10)
    if err != nil {
        return
    }
    defer c.KillRemove()

    url := fmt.Sprintf("postgres://%s:%s@%s:%d/", dockertest.PostgresUsername, dockertest.PostgresPassword, ip, port)
    db, err := sql.Open("postgres", url)
    if err != nil {
        return
    }

    defer db.Close()
    // ...
}
```

Thanks to our sponsors: Ory GmbH & Imarum GmbH
