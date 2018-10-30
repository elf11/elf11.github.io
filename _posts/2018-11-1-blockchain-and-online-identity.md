---
layout: post
title: Blockchain and Online Identity
description: "Blockchain and online identity, ACM XRDS article"
<!-- modified: 2015-09-10 -->
tags: [python, blockchain, 	online identity, cryptocurrency, ACM, ACM XRDS]

comments: true
share: true
---

This article has appeared in a modified form in the Winter issue of ACM XRDS magazine in 2018. There it was a joint work with another author but here is my part of the article. All credit goes to me for this one, if you want to read the joint work you can do so on the ACM XRDS website.<br/>

Today our online identity - which includes our online presence on social media, personal phone numbers and email addresses that are usually used in two factor authentications, credit card numbers, home addresses, passwords and any other personal details is our presence on social media, dictates almost every aspect of our life. But having such a big online presence comes with a lot of fear as well, fear that any security conscious user will experience. This fear is that the huge amount of online data we each own can be abused by distant third-parties is constant. <br/>

We are living at a time of unprecedented concern over identity and what that means to each of us. We base our identity and our identity politics on our online presence today more than ever and it has become a central point of our lives. It’s in this context of constant fear of another security breach, of giant companies who have access through a central point to all of our data, the fear of another leak of emails and passwords from one of the giant tech companies that the blockchain technology has appeared. And, while there might not be a lot of applications that are employing this technology beyond the cryptocurrencies’ universe, one of its future central application looks to be the protection of our online identities and data.<br/>

The easiest way to use blockchain in the area of security and data privacy is quite simple: the data is stored in encrypted form on a decentralized network, and the user can grant other parties access to some of this data by using private keys. So the promise of blockchain in security is to place the control over the data back into the user’s hands.<br/>

So the theme is decentralization, we want to avoid scandals and security breaches like Cambridge Analytica to happen. We have to create and allow smaller communities to be formed online and not limit our online identity through one or two social media profiles. These smaller communities must be allowed to be separate from each other if needed and people must be able to express themselves as individuals. It must also be secure enough for people to trust and limited in scope and reach to form smaller tight-knit communities. Blockchain technology looks to be fitting all those requirements, but is really a good viable option for the future?<br/>


<b>What is Blockchain anyway?</b><br/>

Everything started back in 2008 when an anonymous person or entity using the name Satoshi Nakamoto released a white paper “Bitcoin: A Peer-to-Peer Electronic Cash System”. This laid the foundation of what later became known as Blockchain. In this paper Satoshi described how to build a peer-to-peer electronic cash system that allows online payments to be sent directly from one user to another without going through a centralized institution (e.g. a bank). This system solved the problem of double-spending with digital money. <br/>

So what exactly is double-spending? Suppose you have Alice and Bob going out for dinner, they decide to split the bill right in half, but Alice just happened to forget her wallet at home so she asks Bob to take care of the bill and she will pay him back later. Now, if Alice would pay Bob in cash, then she will physically have to give Bob $30, her half of the bill. After that she will no longer have that money. But if Alice and Bob are using digital money, then the problem gets more complicated. If Alice sends Bob a digital file worth $30, then Bob has no way of knowing if Alice has deleted the file from her records or if she has reused it by sending it to Eve as well. This is called double spending. <br/>


<figure class="center">
	<a href="/images/doubleSpending-blockchain.png"><img src="/images/doubleSpending-blockchain.png" alt="Figure 1. Double Spending Problem"></a>
</figure>

<br/>

One way to solve the double-spending problem is to have a trusted third party between Alice, Bob and Eve, the way that nowadays banking systems are working. This third party is responsible to manage a centralized ledger (a log of transactions) that keeps track of and validates all the transactions in the network. The drawback for this is that for it to function it requires trust in the centralized third party.
<br/>

Blockchain solved the trust problem, there is no need to trust a centralized third party when everyone on the network can verify that your transaction is legitimate. So basically, Blockchain is a technology to create a distributed database and update it without a central authority. The initial Blockchain created was Bitcoin ledger which has the following characteristics:<br/>
<ul>
  <li>Distributed: the ledger is replicated across a number of computers, rather than being stored on a central server; this means that any computer with an internet connection can download a full copy of the blockchain
</li>
  <li>Cryptographic: cryptography is used to make sure that the sender owns the goods that she is trying to send, and to decide how the transactions are added to the blockchain.
</li>
  <li>Immutable: the blockchain can only be changed in append only fashion, this means no past transactions can be altered
</li>
	<li>Proof of Work: in this particular case the Bitcoin ledger has a special characteristic, that miners or nodes in the network compete on searching for a solution to a cryptographic puzzle, this will grant them another bitcoin. </li>
</ul> <br/>
So you can think of a blockchain as a database, the database is made out of blocks, each block encapsulates a number of records. The integrity of the records and implicitly of the blocks is assured using cryptographic keys.<br/>

So how would the problem of double-spending be solved using blockchain? We have Alice that owns Bob money for dining out. Alice, generates a wallet which contains a pair of private and public keys. She wants to pay Bob what she owns him, so she is going to creates a transaction in which she enters both her and Bob’s public keys, as well as the amount of money she wants to transfer to Bob. To make sure that people will know that she is who she says she is, she will sign the transaction with her private key. This way any computer on the blockchain can use Alice’s public key to verify that the transaction is authentic and it will add it to a block that will later be added to the blockchain. In this way there is always a public record of the existing transactions on a ledger, without compromising the identity of the end user - to everyone else you are only identified by a pair of cryptographic keys. The workings of this process can be seen in Figure 2.

<br/>
<figure class="center">
	<a href="/images/Figure2-blockchain.png"><img src="/images/Figure2-blockchain.png" alt="Figure 2. Processing a transaction on Blockchain"></a>
</figure>

<br/>
<br/>

<b>How can you create a private blockchain?</b><br/><br/>

Blockchain is technical concept and can be implemented in variety of ways.There are a multitude of flavors of blockchain. If you are going online and you are looking for cryptocurrencies - the most common implementation of blockchain, then you are going to see around 2000 flavors of cryptocurrencies that are being traded at this point. Some of them are very famous, like Bitcoin or Etherum, others started like a joke, Dogecoin, and then you have a lot of them that are just unknown to the mainstream person. But the scope of this article is not cryptocurrency, it is the technology behind it, blockchain. Can you start your own blockchain? Can you use it at this point for anything? Sure thing you can, you can write your own blockchain in Python.<br/>

As we explained previously, a blockchain is a distributed database with a set of rules that are tracking and verifying new additions to it. In database language, those changes/additions are called transactions, so we need to create an incoming pool of transactions, a way to verify them and add them to a block. <br/>

We are using a hash function to fingerprint each of the transactions and at the same time the function will link our blocks to each other. You can use any hash function from the Python library (sha256 is the one that we used). Next, we want to generate exchanges between the users on the blockchain. We are encoding withdrawals with -1 and deposits with +1. The makeTransaction function in Listing 1 takes care of that, between a sender and a receiver. We are not taking care of the case when we are overdrafting an account here, but that should be trivial. By construction, this makeTransaction  function respects the rule of token conservation, token sent are equal with the token received. <br/>

<br/>
<figure class="center">
	<a href="/images/Listing1-blockchain.png"><img src="/images/Listing1-blockchain.png" alt="Listing 1. makeTransaction function"></a>
</figure>

<br/>
<br/>

We can create a large number of transactions of this type and then group them into blocks. The transaction pool can be just a Python dictionary. From that pool we select k transaction and group them in blocks after we check the validity of the transaction. Remember from Figure 2, that we need to check the signature of the transaction before we are able to attach it to any block. For Bitcoin for example, the validity check is done by checking that the input values are valid unspent transactions, that the outputs of the transactions are not greater than the input, and that the keys used for the signatures are valid. We are going to have a much simpler verify function, we are only checking that the sum of the deposits and withdrawals is 0 (tokens in equal tokens out) and that a sender account has enough tokens in to cover the withdrawal. If either of those conditions are not true then we are rejecting the transaction. The function that does exactly what we described can be seen in Listing 2.<br/>

<br/>
<figure class="center">
	<a href="/images/Listing2-blockchain.png"><img src="/images/Listing2-blockchain.png" alt="Listing 1. makeTransaction function"></a>
</figure>

<br/>
<br/>

Now that we have a way to make a transaction and to verify it, we can create blocks that later will be added to the chain. You can set up your blockchain any way you want, create a couple of accounts for your user, and give them initial credit in a certain number of tokens. Then for each block you will collect a number k of transactions, create a header of the block, hash it, and add it to the chain. To keep the blockchain consistent you will also have to define functions that check that the new blocks you are creating are valid and that the whole chain is valid. And pretty much that’s it, you’d have your own blockchain implementation in Python.

<br/><br/>
<b>Where are we going from here?</b><br/>

We have seen how blockchain works and we also attempted to sketch out an implementation for it in Python. Now, is this a real technology or is there a lot of hype around it? Of course, most people know blockchain from cryptocurrencies, and those have been surrounded by a lot of hype from their inception. People have been riding waves of low and high exchange rates, as well as rapid market crashes that wiped out most of the fortune of crypto-millionaires. Considering all of this, the conversation about blockchain should not be limited to cryptocurrencies. The security and data industries can benefit tremendously from it. The use of blockchain technology grants newfound control the the user. They will be empowered to share their ID data only with those parties they approve. This will be achieved through the use of “decentralized identifiers”. Those identifiers are not only encoding information that are identifying someone who is female, Caucasian, 42, and living in Pennsylvania, but they also make obsolete the need for a central authority to verify IDs. An user will register their ID on a blockchain and then will use their private keys to decrypt data from third parties of interest. In that sense, blockchain definitely has the possibility to revolutionize the world.
<br/>

On the other hand, as with cryptocurrencies people will find a way to use the technology in nefavourious ways, money laundering or buying banned substances on the black market. So, there should always exists a system of check and balances that controls the use cases. Inherently the technology is amazing, but it can always be used in wrong ways.
<br/>


<br/><br/>
<b>Disclaimer</b><br/>
This work is set to appear in the [ACM XRDS magazine][xrds], the Winter 2018 issue. The article in the ACM issue is a joint
work with another co-author. Here I've posted the version of the article that I've submmitted personally to the
magazine. Credits for this article belong to me, disjoint from the magazine version.

Let me know what you think!

[xrds]:https://xrds.acm.org/
    "ACM XRDS"
