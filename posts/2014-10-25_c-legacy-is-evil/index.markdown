---
tags: [c, cpp, thoughts]
summary: >
  A short story about C legacy in C++, and why it's a bad thing to have.
aliases: /2014/10/25/c-legacy-is-evil/
---

C Legacy Is 😈 Evil
===================

When people ask me «What is the first thing I don't like in C++?», I
always answer that's a C legacy. I know a lot of C++ bottlenecks, but I
believe that the worst of them is the C legacy.  What do I mean by the
«C legacy»? I mean all this stuff that doesn't fit into C++ ideology and
kept in the language for compatibility reason. It was a great advantage
years ago, and it's a worst drawback today.

I have a story as a good example of what I'm talking about. When I was
working at [Gameloft], I was involved in [Blitz Brigade] project. In the
very late 2013, the HQ decided to revive Android port and I was chosen
to help Android guys to run both server and client sides of the game.

One day an Android guy asked me for help with debugging. His client
application was rejected by the server and he couldn't figure out why
it's happened. Diving in the code, I have found a typical mistake often
occurring in the C world. The interesting thing was that the mistake was
widespread and should have affected both iOS and Android versions.
However, there was 100% reproduce rate for Android version, while for
iOS it haven't been found yet.

Well, let's look at this typical C code that converts a datetime to Unix
timestamp:

```cpp
// year, month, day, hour, min and sec are retrieved from
// network and have valid values

struct tm tm_struct;

tm_struct.tm_year = year - 1900;
tm_struct.tm_mon = month - 1;
tm_struct.tm_mday = day;

tm_struct.tm_hour = hour;
tm_struct.tm_min = min;
tm_struct.tm_sec = sec;

tm_struct.tm_wday = 0;
tm_struct.tm_yday = 0;

time_t time = std::mktime(&tm_struct);
```

So what's wrong with this code? In first look - nothing is wrong, but for
some reason the resulting `time` variable was `0` on Android and a valid
timestamp on iOS. If you come from C world you probable suspect that the
issue is that the `tm_struct` variable isn't cleared before usage. In C
world we have an unspoken rule:

> Always use `memset` onto struct variable before usage, because by
> default it's filled with garbage.

Example:

```cpp
struct tm tm_struct;
memset(&tm_struct, 0, sizeof(tm_struct));
```

The same rule is also applied in C++ world if we're talking about C
structs, not C++ ones which might be filled properly by means of
constructors. But the `tm_struct` is a C struct, thus it knows nothing
about constructors and we have to clear it manually.

Actually, the idea is not *to clear*, but *to initialize* all members.
It seems like we do initialize all `tm`'s members, but we are actually
not. Unfortunately, the `tm_struct` has one more field - `tm_isdst` -
which is still filled with garbage and leads to mistaken result.

Ok, so why then it works on iOS most of the time and always fails on
Android? I don't know, it just happened and that's all. I think it's
happened because of compiler that may add some clearing code for us
automatically, but I may be wrong.

Why is it so dangerous? Why is there a huge pitfall for C++ in my
opinion?

I believe that C and C++ are different languages with different
ideologies, but with similar syntax. If you wrote your own struct in
C++, you have to define constructor to initialize all members with some
default values. Thus C++ developers don't expect `memset` right after
struct definition, they're relying on constructors which do that thing
for them.

In real world applications built for different platforms using different
compilers it may be a big challenge to debug because the bug may be
floating, so you spent a lot of time to catch it.

So you should be mindful writing programs in C++, since you can fall
into deep hole of C legacy.


[Gameloft]: http://www.gameloft.com/
[Blitz Brigade]: https://itunes.apple.com/us/app/blitz-brigade-online-multiplayer/id580175049?mt=8
