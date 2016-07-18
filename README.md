## What is it?
The FirePrime framework is a [FLOSS][FLOSS] library for your licensing and registration needs in your paid apps. If you plan on using it and want optimum security, read on carefully. If you just want it to work without understanding, which ***will*** lead to issues, scroll down to « How to use it ? »

## How does it work ?
### Theory
FirePrime uses asymmetric cryptography. « Asymmetric » means that there is two different keys : the secret key, and the public key. The secret key, however, must remains *secret*, as it is used to issue licenses. From it is generated the public key, which is the only one required for verifying a license – thus, it should be embedded in all your apps. This asymmetric crypto provides you with excellent security – up to today’s standards : breaking the [Ed25519][ed25519] elliptic-curve algorithm used here is akin, in difficulty, to breaking RSA with \~3000-bit keys. As such, it is computationally infeasible for an attacker to generate fake license files despite the entire framework being open-source – as long as your secret key remains so. 
### Implementation
Practically, FirePrime uses the crypto primitives from the [libsodium][libsodium] library, directly included within the main executable to avoid dylib hijacking and make swizzling the crypto functions harder. Libsodium is widely used, even by companies. ([See who uses it][libsodiumusers]).

[FLOSS]:https://en.wikipedia.org/wiki/Free\_and\_open-source\_software#FLOSS
[libsodium]:https://github.com/jedisct1/libsodium
[libsodiumusers]:https://download.libsodium.org/libsodium/content/libsodium\_users/
[ed25519]:https://ed25519.cr.yp.to

## What should I be wary of ? 
All this nice crypto does not guarantee you that your shiny new app will be uncrackable… because *there is no such thing*, and anyone telling you otherwise is lying. Our goal is to make everything painfully harder, painfully slower, painfully tedious, and painfully painful for the cracker.
So, here you are, you’re done writing your app and want it secure against stealer and crackers and malevolent script kiddies – so you pick up FirePrime and integrate it into your app and publish your app on all & every possible marketplaces and AppStores and whatnot. Obviously, your app is soooo great (congrats !) that an attacker wants to crack it, for your app is soooo great that everyone should be able to use it without giving you back a penny (or whatever his motivation is). Let’s call that bad guy Mallory. In the cracking process, Mallory has two great ways to go : static analysis and dynamic analysis. Static analysis if performed on the binary without running it, while dynamic analysis is done on the program as it is executing. Depending on his motivation and knowledge, Mallory will use one of the following lines of attack :
* __The public key__
	Remember ? It is stored in your app. You have to retrieve it and pass it to FirePrime when instantiating it, before you feed it a license to validate. But Mallory might try to find the public key in you app’s binary and replace it with his own one, of which he would know the private key. Thus, he is able to generate licenses that FirePrime will happily validate because they match the (evil) key you retrieved.
	_Mitigation_ : 
	* First, it is a good thing to hide the key, so that a static analysis would not reveal it straight ahead (it would be really easy for Mallory to swap it for his own one). For example, break it in many smaller parts and rebuild it.
* __The blacklist__
	Which you can optionally use. It’s all up to you, but once again take care because Mallory could modify the list, e.g. removing the hash matching the blacklisted license he wants to use, to have FirePrime return the license as valid.
	_Mitigation_ : See above, it’s no different from the public key.
* __FirePrime itself__
	Because Mallory is so evil that he could hijack (« swizzle ») FirePrime’s validation function, and have it always return the license as valid. Or even replace the crypto functions embedded in FirePrime by his own, corrupted ones. 
	_Mitigation_ : 
 	 * Compile FirePrime as a static library, that will get included in your main binary (automatically, you’ll have nothing to do). By compiling as a shared/dynamic library, you’re risking that Mallory writes one with exactly the same symbols (=functions and classes, if any), and replaces the real FirePrime by his mock – obviously returning what he wants him to. 
 	* Checking the integrity of the binaries is a first step, although not a real obstacle for Mallory if he has enough experience.  
 	* You should also change the integers for each member of the `FPLicenseState` enum (making it harder for the hacker to know the meaning of each number) or change the name of FirePrime’s methods and classes to innocent-looking ones, like « NSThreeRowDictionary » (or « CHOCOLATE »).
 	* You may want to drop Objective-C altogether for license checking, using the plain interface (but leaving the ObjC classes, just as a funny honeypot to lose some of Mallory’s time and erode his patience). Or you might use ObjC’s dynamic nature to your advantage, looking up classes and methods at runtime (storing their names scrambled to keep them away from prying eyes), forcing Mallory to go down the dynamic way (while your app is running), which is harder than a static (non-running) analysis of your code. 
 	* You may also verify the license once, then flip a bit somewhere to make it invalid and verify this one – if the latter check returns true, then you’ll know the evil Mallory is lurking JUST THERE BEHIND YOU BE CAREFUL. (Sorry for the sweat, all those hacker-y babble makes me slightly paranoid at times). And once you know that, it’s all up to you and your imagination ! Most important rule, however: Do *not* taunt Mallory. He and his kind are usually idealists, and this wouldn't discourage them the least. If you challenge them too well, they'll not give up, no matter how hard you make it for them, just to teach you a lesson in return. It'd be a question of honor and pride for them. Next, you might be tempted to take immediate "action" if you detect an attack. Resist from this, for if the cracker sees your app react to the detection of his intrusion, he'll look for that code and disable it. Don't make it *that* easy. Instead, let the attack go unnoticed in your app's behavior first. For instance, modify just a innocent looking variable or pointer that only much later will be accessed again, then leading to an inexplicable crash or other misbehavior of the app.
* Your own code, dammit !
	Because instead of tinkering with FirePrime, he could just plainly remove the calls your code makes to it. But first he’d have to find them, so, once again, use innocent looking names, call from unlikely and sparse places, etc… (read above)

## How to use it ?
So you still want to ? OK, welcome aboard. 
1. `git clone` the repo matching the language you’re using :
	* Plain C
	* Objective-C
	* Swift [upcoming]
	* You name it. Just write an implementation and send me a PR ! Specifications to generate FirePrime-compatible licenses are here : [https://github.com/perfaram/fireprime-protos]
2. Follow the instructions given in each repo. As much as possible, I recommend to keep FirePrime’s code as a git module (for easy updates) and not to use dynamic linking (that is, compile FirePrime as a static library, not a shared one). See above if you don’t understand why.
3. Then read above to know what best practices you should follow. Chances are, otherwise you’ll end with just one call to FirePrime’s `verifyLicense`, which adds virtually no security against crackers.

--------
###### Published in the hope that it will be useful
