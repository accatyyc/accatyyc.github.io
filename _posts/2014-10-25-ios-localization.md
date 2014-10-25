---
layout:     post
title:      iOS Localization
date:       2014-10-25
categories: iOS
---

This post contains some of my thoughts and experiences from localizing iOS applications.

##Format specifiers

After many mistakes when localizing and having to change stuff later I've arrived at the conclusion that you should use the string format specifier (`%@`) for *everything*.

The following example should give you an idea of why.

{% highlight objc %}
NSString *format = NSLocalizedString(@"I have %d red candies.", nil);
NSString *result = [NSString stringWithFormat:format, 5];
// "I have 5 red candies".
{% endhighlight %}

There are two problems with using integer format (`%d`) here:

1. Sooner or later we might want to put something other than an integer in there, like spelling out "five" or using a fractional number like 5.5. 

2. Some languages don't use decimal integers like we do. A  language could theoretically use base 4, or use [different characters](http://en.wikipedia.org/wiki/Eastern_Arabic_numerals) for numbers. I'll explain how to deal with the number formatting in a second.

Had we used `%@` instead of `%d`, both of the above problems could be solved without ordering new, specific translations.

##Combining strings

Combining different localized strings is something I've seen happen every now and then.

For instance, if we already have localizations for the strings `@"Eat"` and `@"Candy"`, it could be very tempting to do something like this to reduce translation costs:

{% highlight objc %}
NSString *eat = NSLocalizedString(@"Eat", nil);
NSString *candy = NSLocalizedString(@"Candy", nil);
NSString *consumeButtonText = [NSString stringWithFormat:@"%@ %@", eat, candy];
// "Eat Candy"
{% endhighlight %}

The problem is that in some languages, the order of words will be inverted. It only gets worse if we combine several words or sentences. I recommend that you spend some more resources on adding all the possible sentences to your localization files, and the translations will be more correct.

##Formatting numbers
Each and every number presented in the UI should be formatted. This guarantees that the application displays numbers correctly even if different locales use different characters or number systems.

{% highlight objc %}
NSNumberFormatter *formatter = [NSNumberFormatter new];
formatter.numberStyle = NSNumberFormatterDecimalStyle;

NSInteger aNumber = 74679;
NSString *formattedNumber = [formatter stringFromNumber:@(aNumber)];
{% endhighlight %}

`formattedNumber` will differ depending on the current locale:

{% highlight objc %}
// en
"74,679"

// sv
"74 679"

// ar
"٧٤٬٦٧٩"
{% endhighlight %}

Since `NSNumberFormatter` is heavy to initialize, I usually keep a few static formatters in a utility class configured for different format styles.

Formatting numbers also have other benefits, for example, not having to write code to round floats or display percentages. If you give a formatter with `NSNumberFormatterPercentageStyle` a float like `0.53467`, it will nicely output `53%`for you. (Or whatever a percentage number should look like in the current locale).

`NSNumberFormatter` really is an amazing class, and I recommend everyone to at least skim through its documentation.

##Autolayout
Autolayout is a great tool to aid us in our quests for perfect localizations. Different languages use different amount of characters to express the same things, and having UI elements automatically make room for text is a real time saver.

It's especially helpful for supporting right-to-left (RTL) languages. Every view that layouts using leading or trailing attributes will be properly laid out together with labels containing right-to-left text.

Most of the time, Autolayout takes care of everything we need for supporting RTL languages. But when something comes up that needs to be done manually (or if we don't use Autolayout at all), we can always branch code like this:

{% highlight objc %}
UIUserInterfaceLayoutDirection layoutDirection = [UIApplication sharedApplication].userInterfaceLayoutDirection;

if (layoutDirection == UIUserInterfaceLayoutDirectionRightToLeft) {
	// Code to handle RTL language
}
{% endhighlight %}

