# Singleton
- much hated design pattern
&nbsp;

> WHen discussing which pattern to drop, we found that we still love them all. (Not really -- I'm still in favour of dropping singleton. Its use is almost always a design smell.) - Eric Gamma

&nbsp;
## Motivation
- For some component it only make sense to have one instance in the system.
  - Database Repository
  - Object Factory
- E.g., Constructor call is expensive
  - We only do it once
  - We provide everyone with the same instance.
- Want to prevent anyone creating additional copies.
- Need to take care of lazy initialization and thread safety.