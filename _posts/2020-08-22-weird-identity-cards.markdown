---
layout:	post
title:	"Weird Identity Cards"
date:	2020-08-22 16:00:00 +0300
categories: API
---
Yestarday, a need arose to create a simple library to verify a person's
identity using their National Identity Cards, (of the Republic of Turkey,
where this story takes place.) to do so, I Google'd a little and stumbled
upon a public API by the relevant government authority. It used an older
system than I used to, but it was working fine, and I was able to create a
small Python library, [PyTCID].

While creating PyTCID, as I read the documentation of the said API, I
realised something strange, the XML input needed for the API had three
strange parameters. `"<NoSurname>"`, `"<NoBirthDay>"` and `"<NoBirthMonth">`.
Intruged, I first assumed these were optional parameters, that I could
verify a identity card by leaving these fields empty, alas, no. If your
card has these fields, you need to provide them, which implies: "Are there
identitification cards without these fields?!"

Yes. Yes, there are.

Starting with the birthday and birthmonth, there are actually is an
intresting [article] about the subject, mentioning a 76 year old man
whose birth month and day is given as 0. This is because, apperantly,
prior to 2006, people with unknown birthdays were given the day 0. I
also heard of anectodes of people seeing cards without birthdays and
months, since the oldest generation sometimes do not know their exact
birthdays, as in their days, rural communuties kept time not using the
Julian callendar, and not even the older Islamic callender, but used
crop cycles, ie: "Born during harvest". Therefore, these people remained
without a set birthday and month, and with only a year.

Sure, but what about cards withotu surnames? Well, First Identity Cards,
or documents that were their direct ancestors can be dates to [1928],
but, Turkish people did not have a surname (at least in the legal sense)
until 1934, which means that people who have died before this time,
technically had a valid identification card, but no surnames, I assume
that is why the API allows checks without surnames.

All in all, something as simple as an identity check API has intresting
historical footnotes in it. Decisions that were perhaps  taken for
backwards compatibility with older systems result in deeper than 
expected rabit holes. It was quite a peculiar experince indeed.

[PyTCID]: https://github.com/ambertide/PyTCID
[article]: https://www.hurriyet.com.tr/nufus-cuzdaninda-dogum-tarihi-yok-37017309
[1928]: https://www.aa.com.tr/tr/turkiye/osmanlidan-gunumuze-turkiyenin-kimlik-kartlari/531988
