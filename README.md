
# cail ("sail")

Cail is a small library for parsing the insane _javax.mail_ library
classes into Clojure data structures.

## Usage

```clojure
(require [cail.core :refer [message->map]])

(def original (.getJavaMailMessageFromSomewhere java))

(def message (message->map original))

(:subject message) ; => "The Subject"
(count (:attachments message)) ; => 3

; etc...
```

When given a message to parse it will still require the connection
to its associated folder to fetch its data from.  The message body
will prefer HTML content types.

## Attachments

Due to the possible (and probable) size of attachment content this
is not extracted by default.  All the metadata for attachments will
be available after parsing a message, but to fetch the content
you need to ask for it.

```clojure
(def message (MimeMessage.))
(def msg (with-content-stream
           (message->map message)))
(def attachments (:attachments msg))

(println (count attachments )) ; => 2

(def attachment (first attachments))

(:content-type attachment) ; => "application/pdf"
(spit "~/file.pdf" (:content-stream attachment))
```

The _:content-stream_ is an input stream for reading the attachment.

### Individual Attachments

You can also ask for individual attachments by index...

```clojure
(def attachment (message->attachment message 0))
```

By default this will *not* return the content stream for the 
attachment, just wrap it in _with-content-stream_ to fetch that.

## Cachability

One goal of Cail is to allow messages to be cacheable. So after
parsing they don't rely on any internal state (connections to
mail stores) that the Java versions do.

This means that after parsing and caching a message the only time
you need to go back to the mail store is to fetch the content
of attachments (which could be many MB's so are not cached).

## Testing

Tests use _clojure.test_.

```
lein test
```

