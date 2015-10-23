# Event-Driven-Programming
- The flow of program is determined by events
- When a event is triggered, perform the pre-defined methods

Let's define event and event handler
```c
  typedef void (*event_handler)(evnet *);

  typedef struct event {
    event_handler  handler;
    void          *data;
  }
  
```
Then we register the event to an event looper
```c#
  
  void consume(event *e) {
    if(e->ready) {
      // do some work
    }
  }
  e = create_event();
  e->handler = consume;

```
The looper routine
``` c#

  register_event(looper, e, fd, flag); // when fd is ready for operation, triggers event e
  
  while (forever) {
    events = loop(looper, interval); // block for time interval specified and return triggered events 
    for (e in events) {
      e->handler(e);
    }
  }
  
```
What if the event never happens?
```c#
  
  e = create_event();
  e->handler = process_event;
  e->timeout = current_time() + timeout;  // set as an absolute moment
  
  add_to_timer(e);
  register(looper, e, fd, flag);
  
  /* looper routine with timer */
  while (forever) {
    e = next_timeout_event();
    interval = e->timeout - current_time();
    interval = interval>0 ? interval : 0; 
    events = loop(looper, interval);
    if (e in events) {
        e->ready = 1;
        e->handler(e);
    }
    
    /* check timeout events */
    for (;;) {
      e = next_timeout_event();
      if (e->timeout < current_time()) {
        del_from_timer(e);
        e->timedout = 1;  // timedout flag is set to inform the event handler
        e->handler(e);
        
      } else {
         break;
      }
    }
  }
    
```
# Asynchronous IO
A form of input/output processing that permits other processing to continue before the transmission has finished.
To realize this, Linux kernel supports non-blocking sockets.

```c
nb = 1;
ioctl(socketfd, FIONBIO, &nb); // set socket as non-blocking mode
```
The io syscall like recv() and write() can now operate in non-blocking mode on the socket.
- If some messages are processed, return the length of the message processed
- If no messages are available at the socket, the value -1 is returned _immediately_ and the external variable errno is set to EAGAIN or EWOULDBLOCK. 
  
``` c
ssize_t nb_recv(connection *c) {
  ssize_t     n;
  ngx_err_t   err;
  
  do {
    n = recv(c->fd, c->buf, c->size, 0);
    if (n > 0) {
      return n;
    }
    
    if (n == 0) {
      c->eof = 1;
      return n;
    }
      
    err = errno;
    if (err == EAGAIN || err == EINTR) {
      n = SOCKET_AGAIN;
    } else {
        n = SOCKET_ERROR; /* recv error happens */
        break;
    }
    
  } while(err == EINTR); // signal interupt, will continue
  
  retirn n;
}
```
So this function can be combined with event-driven model.

```c
event *e;
connection *c
e = create_event()
c = open_read_connection(fd);

e->data = c;
e->handler = consume;
register(looper, e);

void consume(event *e) {
  ssize_t  n;
  connection *c;

  c = (connection *) e->data;
  if (e->timedout) {
      process_timeout(c);
      return;
  }

  n = nb_recv(c->fd, c->buf, c->size);

  if (n>0) {
  
    process_buf(c);
    
  } else if (n==0) {
  
    process_eof(c);
    
  } else if (n==SOCKET_AGAIN) {
  
    e->timeout = 5000+current_time(); // 5000 ms
    add_to_timer(e);
    register(e);
    
  } else {
    process_error();
  }
}
```
  

