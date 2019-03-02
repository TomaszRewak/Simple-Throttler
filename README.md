# Simple-Throttler
A simple C++ throttler implementation.

This throttler supports sending messages from multiple users. Each user can send only a certain number of messages within the period defined by the sliding window.

Basic usage:
```cpp
auto throttler = make_message_throttler<UserId, Message>(
  max_number_of_messages_per_user,
  sliding_window_width,
  message_consumer
);

throttler.from(1).send("Message 1");
throttler.from(1).send("Message 2");

throttler.from(2).send("Message 3");
```
or
```cpp
auto throttler = make_message_throttler<UserId, Message>(
  max_number_of_messages_per_user,
  sliding_window_width,
  message_consumer
);

auto& first_client_interface = throttler.from(1);
first_client_interface.send("Message 1")
first_client_interface.send("Message 2");

auto& second_client_interface = throttler.from(2);
second_client_interface.send("Message 3")
second_client_interface.send("Message 4");
```

To be more specific:
```cpp
auto throttler = make_message_throttler<int, std::string>(
  4,
  std::chrono::milliseconds{ 1000 },
  [&](const std::string& message) { std::cout << "Consumed: " << message << endl; }
  [&](const std::string& message) { std::cout << "Disposed: " << message << endl; }
);

throttler.from(1).send("Hello");
```

Properties:
- I've separated single client interface from the main throttler. Thanks to this if one client wants to send multiple messages only one map lookup is required.
- Throttler supports custom message types and client id types.
- Complexity of the `send` method (of the client interface) is guaranteed to be O(1) no matter how long the sliding window is and how many messages are already registered in it. Also, after sending N (= sliding window size) messages the list of instructions performed to send [dispose] messages is the always the same (which should make it easy for branch prediction algorithm).
- `send` method (of the client interface) never allocates.
- User can define his own data consumers (any collable functor is ok). He can also listen for discarded messages.
- By default (defined in `message_throttler_commons.hpp`) throttler uses chrono for timestamping. This behavior can be overwritten with any custom timestamper and sliding window processor.
- Calling the throttler's `from` method hashes the clientId only once. Map lookup is also performed only once, even if given client has not been registered yet.

By default (defined in message_throttler_commons.hpp) throttler uses chrono for timestamping messages. This behavior can be overwritten with any custom clock.

This is a header only component. Apart from standard libs, it depends on boost's circular_buffer.
