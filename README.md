# Whisper Addresses
Privacy preserving automatic bitcoin address generation in vanilla javascript

# Testnet Demo

View a demo of how whisper addresses work here: https://supertestnet.github.io/whisper-addresses/demo.html

# Mainnet Demo

View a demo of how whisper addresses work here: https://supertestnet.github.io/whisper-addresses/demo-mainnet.html

# Video demo

Sending the bitcoin: https://twitter.com/PlebLab/status/1507217415691722752
Withdrawing the bitcoin: https://twitter.com/PlebLab/status/1507222897886216193

# Intro

Whisper addresses are bitcoin addresses that are generated by means of point addition. Point addition will be described in a moment, first I want to share the "why."

# The why

Imagine you want to create a website to help protesters receive bitcoin in a country where protesting is illegal. You might create a spreadsheet that has the names of some of the protest leaders and their bitcoin addresses, then put that spreadsheet online so that people can find it easily and make their donations. One of the problems with that approach is that the donors aren't the only people who will see that page. The country's authorities could find the site too and order exchanges in their country to blacklist those addresses -- "if they try to cash out that bitcoin, confiscate it!"

This is where whisper addresses are helpful. Whisper addresses allow you to put something called a "linking key" into the field where a bitcoin address would go. Linking keys allow each visitor to your site to generate a bitcoin address that "belongs to" the person who created the linking key. Each visitor will generate a "whisper" bitcoin address which no one can associate with the linking key except the person who generated it and the recipient of the money. Each whisper address will start out empty. If the police visit the site, they will see an address that no one has ever seen before and which will likely never be seen again after the police leave the site. They can blacklist that address, but there's no money in it so it's no big deal. Each "legit" visitor will also generate a whisper address that belongs to the intended recipient. These whisper addresses will also be things that were never seen before, and therefore they are not on the police's blacklist. If the user sends money to it, they can also send some data to the intended recipient that allows them to find the address on the blockchain and withdraw the money in it to their regular wallet.

In short, the "why" of whisper addresses is censorship resistance. Whisper addresses are unique for each visitor to the site, cannot be associated with the recipient, and cannot be blacklisted by exchanges.

# Are whisper addresses related to BIP47?

BIP47 is a bitcoin improvement proposal that defines a protocol for generating and using things called stealth addresses. These are almost identical to whisper addresses with one big differences: stealth addresses have a property called determinism. Determinism means that stealth addresses have an associated "marker" which allows the recipient's wallet to discover that stealth address by scanning the blockchain. The wallet can also use the marker to automatically derive the private key they need to withdraw money from the stealth address, but anyone other than the intended recipient cannot derive the correct private key. This determinism property makes usage of stealth addresses easy for recipients.

But determinism comes with a cost. In order to achieve determinism, the person sending the money has to put an additional transaction on the blockchain containing the marker in a format that the recipient's wallet can uniquely decode. That makes these transactions expensive. Also, bitcoin wallets that do not understand the BIP 47 protocol also cannot send money using it, and developers cannot implement a javascript interpreter to "convert" BIP 47 linking keys to regular bitcoin addresses because they can't put the marker on the blockchain without controlling the user's funds. (The marker is not "in" the bitcoin address, it's in some additional data separate from the bitcoin address but which is supposed to be added to the blockchain via a special bitcoin transaction called an op_return.)

Non-deterministic stealth addresses ("whisper addresses") are described by this protocol. They differ from BIP 47's "deterministic" stealth addresses because the marker is not added to the blockchain, and thus the recipient's wallet cannot automatically detect and decode them. Instead, the "marker" is essentially sent to the recipient in an email or via some other out-of-band communication method. (Actually whisper addresses don't use a "marker" but rather something called an ephemeral private key, but I think of it as the non-deterministic equivalent of a marker.) Sending data out-of-band instead of adding it to the blockchain reduces costs and allows developers to create javascript interpreters which can display a regular bitcoin address to the payer without his wallet needing to know how the protocol works.

# The How: Point Addition

Bitcoin addresses have two main components: a public key and a private key. The private key is something you're supposed to keep secret because it lets you move the money in the bitcoin address. The public key is fine to share with people, it's practically the same thing as a bitcoin address but with a special encoding so that bitcoin wallets can recognize them and to make them distinct from public key cryptography systems other than bitcoin. You're supposed to share your bitcoin address with people in order to receive money from them and it's also fine to share the underlying public key.

Public keys and private keys are both made of numbers. Imagine a simplified public keys system that works like this:

```
123 <-- that's a public key
ABC <-- that's a private key
```

```
456 <-- that's another public key
DEF <-- that's another private key
```

One of the things you can do with public key cryptography is called point addition. Point addition allows you to add two public keys and two private keys together and still get a valid keypair. For example:

```
142536 <-- that's 123 and 456 added together
ADBECF <-- that's ABC and DEF added together
```

Those are both still perfectly valid public keys and private keys, and moreover, the "combined private key" is capable of spending money that someone sent to the "combined public key."

Whisper addresses take advantage of the point addition property to enhance the recipient's privacy. The recipient of the money can create a public key (such as 123), put it on a public website, and keep the private key secret. The police may blacklist his public key but that's fine because, as you'll see in a moment, he won't actually receive any money at that public key. Instead, it will only serve as a "linking key" which other people can use to send money to other bitcoin addresses that are not blacklisted but in such a way that the recipient can still recover the money in those addresses.

Users who want to use these linking keys just have to visit the public website. Their browser will generate a fresh keypair, which I call an ephemeral keypair. (In the example above, 456/DEF might be an ephemeral keypair.) Their browser also automatically adds their ephemeral pubkey to the recipient's linking key and thus derives 142536 -- the whisper address -- which it displays to the visitor. Each visitor to the site will see a different whisper address because their browser will generate a different ephemeral keypair. After they send money to the whisper address, they can email their ephemeral private key to the recipient of the money. He can then add the ephemeral private key to his linking key's private key and thus derive the private key he needs to take money out of the whisper address. Voila! He received money to a unique address that is not on anyone's blacklist, which is very censorship resistant, and the people visiting his website did not need a special wallet that knows the protocol and also did not need to add a special marker to the blockchain and thus incur an unnecessary expense.

# Summing up

Whisper addresses are a useful addition to the bitcoin privacy toolset. They allow greater censorship resistance when receiving payments but at a lower cost than BIP 47 stealth addresses, with less blockchain bloat, and without requiring bitcoin wallets to implement additional software.

# TLDR

* Whisper addresses are bitcoin addresses that are created using point addition by two or more people working together 
* Only one of those people can withdraw money from the whisper address, and exchanges or governments can't know who it is
* Point addition works like this: take two or more public keys and add them together to get a new pubkey -- the whisper address
* The only way to get the whisper address's private key is to add together all the original private keys
* The money "senders" and the recipient create the whisper address, and the senders give their private keys to the money recipient
