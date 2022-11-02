# How to create a project
The contaneir created by this Dockerfile does not install Drupal. It assumes you have an existing project, but if you don't you can create one easily using a composer image.

```
podman run --rm -v ./:/app:Z composer composer create-project drupal/recommended-project test --ignore-platform-reqs
```

This will create a new directory called test under your current directory. If you aren't using a labeling system, such as SELinux, you can ommit the `:Z` in the `-v` option. In that new directory called test, Drupal will be downloaded. Notice the addition of the `--ignore-platform-reqs` option. This is required because the composer container does not meet the requirements for Drupal. You must ensure that the Docker file in this directory meets the php requirements for your project, and if it does not, then update it to add your dependencies/libraries.

# How to add new dependencies to your project
In a similar fashion, if you want to add new dpendencies, or, for that matter, do anything that requires running composer, you need to use a composer image.

Firs, cd to your project's directory
```
cd test
```
Then run the require command
```
podman run --rm -v ./:/app:Z composer composer require 'drupal/pathauto:^1.8' --ignore-platform-reqs
```

Note that while we are using podman in these examples, it should work the same using docker.

