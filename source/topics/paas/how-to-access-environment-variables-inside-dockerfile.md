By design (for better or worse), Docker doesn't allow setting any environment variables during the Docker build process, only afterwards on running containers instantiated from the built images. The idea is that images should be fully portable and not tied to any specific environment. This means that your Aptible `ENV` variables, set via `aptible config:set`, are not available to commands executed during your Dockerfile build.

Our workaround for this is to write a special `.aptible.env` file to your app's repo immediately before building the app from its Dockerfile. The `.aptible.env` file will contain your current configuration key and value pairs in a format that can be sourced into your shell. For example, your `.aptible.env` file might look like:

```
KEY1=value1
KEY2=value2
```

To source your app's current configuration for a command — say, `bundle exec rake assets:precompile` — you could use the following line in your Dockerfile:

```
# Assume that you've already ADDed your repo:
# ADD . /opt/app
# WORKDIR /opt/app

RUN set -a && . /opt/app/.aptible.env && bundle exec rake assets:precompile
```

Note: The `quay.io/aptible/autobuild` base image will automatically load `.aptible.env` before building your app, so it is not necessary to explicitly source the file when using that image.