# event-driven-programming
- The flow of program is determined by events
- When a event is triggered, perform the pre-defined methods

Let's define event and event handler
```c
  void (*event_handler)(evnet *);

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
      e->ready = 1;
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
  

