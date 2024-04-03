# Exchange

ExchangeFinder
# Gmail

Abusing `/gxlu` for user enumeration

https://blog.0day.rocks/abusing-gmail-to-get-previously-unlisted-e-mail-addresses-41544b62b2

Script it.

tl;dr - `https://mail.google.com/mail/gxlu?email=<valid account>` 204 response has a `Set-Cookie` header that an invalid account doesn't.

This should still work as it has been wontfix for years, but I haven't tried it in a while.




