# Relé

[![Build Status](https://travis-ci.org/mercadona/rele.svg?branch=master)](https://travis-ci.org/mercadona/rele)

Relé makes integration with Google PubSub easier and is ready to integrate seamlessly into any Django project.

## Motivation and Features

The [Publish–subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) and specifically the Google Cloud [PubSub library](https://pypi.org/project/google-cloud-pubsub/) are very powerful tools but you can easily cut your fingers on it. Relé makes integration seamless by providing Publisher, Subscriber and Worker classes with the following features:

* A Publisher:
  * Singleton: Ensures there is no memory leak when instantiating a `PublisherClient` every time you publish will result in a memory leak because the transport is not closed by the Google library.
  * Automatic building of topic paths
* A `sub` decorator to declare subscribers:
  * Super easy declaration of subscribers
* A Worker:
  * In-built DB connection management so open connections don't increase over time
* A `python manage.py runrele` management command
  * Automatic creation of Subscriptions
  * Subscription auto-discovery

## What's in the name

"Relé" is Spanish for *relay*, a technology that [has played a key role](https://technicshistory.wordpress.com/2017/01/29/the-relay/) in history in the evolution of communication and electrical technology, including the telegraph, telephone, electricity transmision, and transistors.

## Quickstart

Add it to your `INSTALLED_APPS`:

```python
INSTALLED_APPS = (
   'rele',
)
```

You'll also need to set up two variables with the Google Cloud credentials:
`RELE_GC_CREDENTIALS` and `RELE_GC_PROJECT_ID`.

NOTE: Ensure that [`CONN_MAX_AGE`](https://docs.djangoproject.com/en/2.2/ref/settings/#conn-max-age)
is set to 0 in your worker. The Django default value is 0.

In other words, the environment where you run `python manage.py runrele`,
make sure `CONN_MAX_AGE` is not set explicitly.

## Usage

```python
# your_app.subs.py

from rele import Publisher, sub

def publish():
      client = Publisher()
      client.publish(topic='lets-tell-everyone',
                     data={'foo': 'bar'},
                     myevent='arrival')

@sub(topic='lets-tell-everyone')
def sub_function(data, **kwargs):
      event = kwargs.get('myevent')
      print(f'I am a task doing stuff with the newest event: {event}')
```

### Subscription `suffix`

In order for multiple subscriptions to consume from the same topic, you'll want to add
a unique suffix to the subscriptions, so they both listen to all the gossip going around.

```python
@sub(topic='lets-tell-everyone', suffix='sub1')
def sub_1(data, **kwargs):
     pass

@sub(topic='lets-tell-everyone', suffix='sub2')
def sub_2(data, **kwargs):
     pass
```

### Running the worker in a process

In your worker, you can run `python manage.py runrele`. Once subscribed to
the topic, in another process you can run the `publish` function. Your subscription process
should print out the message.

----

## Running Tests

Does the code actually work?

      pytest
