Mongoom
=======

Release v0.1.1.

Stay Pythonic while working with MongoDB. Mongoom provides a light-weight api for mapping MongoDB documents to Python objects on top of pymongo.

Features
========

- Encoding and Decoding of documents.
- Active Validation
- Document based Events
- Threaded Subscriber


Using Mongoom is simple!
========================

Inherit from Document and EmbeddedDocument to define a schema.

::

    from mongoom import *


    class User(Document):
        name = Field(basestring, required=True)
        last_name = Field(basestring, required=True)


    class Comment(EmbeddedDocument):
        user = Field(User, required=True)
        text = Field(basestring, required=True)
        created = Field(datetime, default=datetime.utcnow)


    class Project(Document):
        name = Field(basestring, required=True)
        user = Field(User, required=True)
        created = Field(datetime, default=datetime.utcnow)
        description = Field(basestring)
        comments = ListField(Comment)


Establish a connection and save some documents.

::

    connect("test_db", "localhost", 27017)

    edison = User(
        name="Thomas",
        last_name="Edison",
        ).save()

    bulb = Project(
        name="Light Bulb",
        user=edison,
        description="Create a commercially viable light bulb.",
        ).save()

    naysayer = User(
        name="Anonymous",
        last_name="Naysayer",
        ).save()

    rude_comment = Comment(
        user=naysayer,
        text=("It's impossible to create a viable light bulb. Like all of"
              "Mr. Edison's ideas, this too will be proven impractical."),
        )

    bulb.comments.append(rude_comment)
    bulb.save()


Retrieve and modify a Document.

::

    bulb = Project.find_one(name="Light Bulb")
    edison = User.find_one(last_name="Edison")
    rebutt = Comment(
        user=edison,
        text="I'll show you!")
    bulb.comments.append(rebutt)
    bulb.save()


Also included with Mongoom is an Event and Subscriber. Event objects are nothing more than a Document object residing in a capped collection. While subscribers are tailable cursors awaiting data to be entered into a capped collection. Using these two objects we can easily create a simple event handling system:

::

    from mongoom *

    class Create(Event):
        '''Create Event'''

    class EventHandler(Subscriber):
        def handle(self, document):
            print document
            print document.ref.data

    connect("test_db")

    fire(Event)  # Fire a blank Event to initialize capped collection

    regret = Comment(
        user=User.find_one(name="naysayer"),
        text="I feel like an idiot, the light bulb turned out great."
        )
    bulb = Project.find_one(name="Light Bulb")
    bulb.append(regret)
    bulb.save()
    fire(Create, ref=idiot)

    ev_handler = EventHandler("Event")
    ev_handler.start()


For a more elaborate mongorm event-driven system check out EventSubscriber.py in examples.

Installation
============

::

    git clone https://github.com/danbradham/mongoom.git
    cd mongoom
    python setup.py install

Documentation
=============
Visit http://mongoom.readthedocs.org for full documentation.
