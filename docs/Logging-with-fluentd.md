# Logging with fluentd

> Reference docs :
> https://www.fluentd.org/faqs

Install via ruby
https://docs.fluentd.org/installation/install-by-gem

Install docker image
https://github.com/fluent/fluentd-docker-image
https://docs.fluentd.org/container-deployment/install-by-docker

To allow writtable log on host
https://github.com/fluent/fluentd-docker-image/issues/2

```
sudo chmod 777 /tmp/log/
```

Logging of Fluentd daemon
https://github.com/fluent/fluentd-docs/blob/master/docs/v1.0/logging.txt