Feature Toggling
==============================

Feature toggling, switching and dark releases sound more mystical than they are. In this section, we discuss the different strategies for available for *decoupling releases from deployments*.

First, we need to answer the following questions:
> What is a release?
> What are deployments?

*Releases:* A release is the act of delivering a piece of functionality to users

*Deployments:* Deployments are act of installing new software and code to a given environment.

There is no reason why the two should be synonymous. A delivery team should not have to push new software versions to a given environment for a piece of functionality to be released to live users. There are risks with doing so:

<ol>
<li>Operational risk: If the code and functionality are being delivered at the same time, it is likely that this is the first time that this code has been installed on this environment.</li>
<li>Intervention: It is also likely that deployments have some element of manual intervention involved, whether it is pushing a button, or approving a change. Businesses on the other hand may prefer to have releases during off-peak hours. Someone has to stay up to perform a midnight/weekend release.</li>
<li>Opportunity cost: Coupled business releases and deployments are a symptom of long lived branches. It is likely that these branches containing feature X are released to production at the expense of another branch containing feature Y.</li>
<li>Stale: See our section on Trunk Based Development. It is likely that long lived feature/release branches which contain feature X could contain commits and changes which need to be contantly rebased into multiple other branches, either leading to merge conflicts and development pain.</li>
<li>Eggs in one basket: It is likely a coupled release and deployment stem from a non-trunk based development strategy which exhibits long lived branches. It is therefore possible to assume that feature X could contain work associated with hundreds of commits, tickets, user stories and requirements. A batch of changes this large is likely to raise the risk profile of your deployment.</li>
</ol>


Feature Toggles
--------------

A simple, yet crude explanation of toggling is the use of conditional logic in your code to show or hide features based on runtime configuration.

An example of this would be:

```
if(welcomeScreenIsActive) {
    showWelcomeScreen();
} else {
    hideWelcomeScreen();
}
```
One thing to note here is that the flag `welcomeScreenIsActive` can have one of the two boolean states. There is a problem with a boolean feature flag approach, and it is this.

To change the value of this flag, and to (de)activate this code, there are two options:
* either your configuration is version controlled for audit purposes, and requires a deployment or some intervention such as restarting the server with updated params to activate in production
* or you are able to toggle the value of this on-the-fly to turn the feature on/off

The trade off here is convenience and flexibility over audit trails. This is where date based flags reign supreme.

Date based features
-------------

The concept here is simple, you can codify logic to ensure you code logic can turn on/off at some predefined point e.g. 

Your configuration file may look like this:
```    
feature.X.enabled = "2020-01-01T00:00:00.000Z"
```
and your code may look like this:
```
    if(LocalDateTime.parse(featureXEnabled).isAfter(DateTime.now())) {
        showWelcomeScreen();
    } else {
        hideWelcomeScreen();
    }
```

