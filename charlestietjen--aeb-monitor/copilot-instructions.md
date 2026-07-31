## project

> -This project uses Deno 2.4.2 and you must use practices, methods and libraries appropriate for this version.


-This project uses Deno 2.4.2 and you must use practices, methods and libraries appropriate for this version.

-The goal of the project is a server that checks the stock of a product at a small number of Canadian Tire store locations using stocktrack.ca - there is no public API for stocktrack.ca so we'll be working based off of network traffic. The following URL is what we'll be using as a reference.

https://stocktrack.ca/ct/availability.php?store=0459,0150,0273,0654,0600,0030,0019,0192,0621,0214,0209,0182,0175,0485,0126,0264,0119,0321,0087,0242&sku=1502321&src=upc

The parameters are self explanatory.

The server should check the url once an hour. If there's stock it sends an email to an email address defined in a config.json we'll load at run-time.

The store's to check will also be defined in the same config.json as well as the product sku.

---
> Source: [charlestietjen/aeb-monitor](https://github.com/charlestietjen/aeb-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
