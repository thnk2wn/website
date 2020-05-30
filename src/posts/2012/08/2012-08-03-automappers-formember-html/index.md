---
title: "AutoMapper's ForMember"
date: "2012-08-03"
---

Calling AutoMapper's ForMember has been bugging me lately with having to deal with its member configuration options like so:  

\[csharp\] Mapper.CreateMap<Contact, AddressBookDetailsModel>() .ForMember( d => d.PhoneNumbers, o => o.MapFrom( s => s.Phones ) ); \[/csharp\]

The member configuration options provides much more than just MapFrom but 99 times out of 100 I'm only dealing with MapFrom. What I'd really like is something like this:  

\[csharp\] Mapper.CreateMap<Contact, AddressBookDetailsModel>() .ForMember( d => d.PhoneNumbers).MapFrom( s=> s.Phones ); \[/csharp\]

In looking around I did find [http://trycatchfail.com/blog/post/A-More-Fluent-API-For-AutoMapper.aspx](http://trycatchfail.com/blog/post/A-More-Fluent-API-For-AutoMapper.aspx). It looked promising but (a) the code was incomplete and (b) it broke down for more complex mappings.  
  

I'd like the syntax above but for now I went with something quicker to implement that gave me a shorter syntax though it might not be as syntactically sweet:  
\[csharp\] public static class AutoMapperExtensions { public static IMappingExpression<TSource, TDestination> MapItem<TSource, TDestination, TMember>( this IMappingExpression<TSource, TDestination> target, Expression<Func<TDestination, object>> destinationMember, Expression<Func<TSource, TMember>> sourceMember ) { return target.ForMember( destinationMember, opt => opt.MapFrom( sourceMember ) ); } } \[/csharp\]

Now when creating maps I don't need to fuss with an extra lambda and method call: \[csharp\] Mapper.CreateMap<Contact, AddressBookDetailsModel>() .MapItem( d => d.PhoneNumbers, s=> s.Phones ); \[/csharp\]

I can do something similar to simplify Ignore calls:  

\[csharp\] public static class AutoMapperExtensions { public static IMappingExpression<TSource, TDestination> Ignore<TSource, TDestination>( this IMappingExpression<TSource, TDestination> target, Expression<Func<TDestination, object>> destinationMember) { return target.ForMember( destinationMember, opt => opt.Ignore() ); } /\* other code removed for brevity \*/ } \[/csharp\]
