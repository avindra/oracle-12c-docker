# Oracle provides this

Oracle has a [canonical repository](https://github.com/oracle/docker-images) for building docker images for many of their products, including [Oracle Databases](https://github.com/oracle/docker-images/tree/master/OracleDatabase/SingleInstance).

# Use cases I want to support

 * Local development databases
 * Databases for use in continuous integration
 * Single instance. RAC deployments are supported, but not necessary for the above two use cases.

# How do I build the image?

Oracle has pretty good instructions and scripts for doing this, but it isn't super clear right away. For instance, since you are using Docker, you probably want to have a [prebuilt db](https://github.com/oracle/docker-images/tree/master/OracleDatabase/SingleInstance/samples/prebuiltdb) as well. Here was the end result of the first iteration in my journey:

```
 $ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
oracle/db-prebuilt   12.1.0.2-ee         00888a40e015        26 minutes ago      14.9GB
oracle/database      12.1.0.2-ee         18553d8c8419        38 minutes ago      10.4GB
```

That's right, the base image turned out to be 10.4GB and the prebuilt image, a whopping 14.9GB for an image with a pre-built database. Try explaining that to your network admin / container team.

Fortunately, Oracle is [aware of this image size problem](https://github.com/oracle/docker-images/issues/896), and suggests a workaround, which is essentially to use the experimental `squash` option of `docker build`. For example, if you are building an enterprise version of 12.1.0.2, you would build it as:

```
 ./buildDockerImage.sh  -v 12.1.0.2 -e -o --squash
```

Note that this is an experimental feature, so you will have to pass `--experimental=true` to `DOCKER_OPTS` in `/etc/sysconfig/docker`.

New image sizes after enabling `--squash`:

```
$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
oracle/db-prebuilt   12.1.0.2-ee         801cde12e07c        3 seconds ago       9.5GB
oracle/database      12.1.0.2-ee         aad4e86f3d3b        5 minutes ago       5.05GB
```

Much better! I suspect the `db-prebuilt` tag can be shrinked down as well.

# Wishlist

Oracle please:

 * Express edition for 12c
 * 18c+ container building scripts
 * Allow removing features, to help reduce container footprint (storage and compute). For example, I should be able to disable APEX which would prevent the files from being installed.
 * Canonical docker hub images - so we don't have to discover the build issues above and everyone can just point to a single source of truth :)
