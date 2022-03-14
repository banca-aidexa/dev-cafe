# Feature Toggles

A Feature Toggle (or Feature Flag) is a mechanism to modify a particular behaviour of your software at runtime with no deploy.

They are often used, for example, to safely deliver a new feature to a restrcited set of your users, to run A/B testing or to activate PRO features to paid users.

## Types of Feature Toggles

Feature Toggles are really powerful and can enable a lot of different behaviours. Let's see the most common.

### Canary release

This technique is usually associated to **deployement**: you may have 4 running pods of your application and you can instruct Kubernetes to replace only 1 of the 4 with the new version of your software.

In this base version, you will have 25% of your new traffic routed to the new Pod, the Canary one and thus inspect logs and see how it is behaving.

You can keep the canary alive as much as you need and then decide to kill it (Kube will deploy a 4th pod with the previous version) or promote the Canary to be the stable deployed version.

#### Why "Canary"?

The name for this technique originates from miners who would carry a canary in a cage down the coal mines. If toxic gases leaked into the mine, it would kill the canary before killing the miners. A canary release provides a similar form of early warning for potential problems before impacting your entire production infrastructure or user base.

### A/B testing

This mechanism is loved by Product Owners and Business: you have the power to have 2 different behaviours running at the same time for two different users segment and **measure** which is "better".

Imagine something that may changer the customer journey, you want to measure which feature (A or B) is improving _conversion rate_ more.

### Release toggles

They are used to separate the moment in which you deploy the code from when you _enable_ the code. You are releasing latent code which is not creating any impact and you may decide to turn it on later (or never).

They are usually handled in a static way, and it is perfectly accetable to enable them with a new release.

You should not keep the flags relevant for more than 2-3 weeks as they may rot.

### Ops toggles

Operational Toggles are vital when you are releasing a new feature whose _performance_ you may not know beforehand or a feature you know is poor performing under some circumstances.

It may be a new refined query on your database or a new API call to a 3rd party partner, you would like to dynamicall turn it off (and go back to previous behaviour) with just a click.
It may even be a particular **log** which is very heavy on disks, you may want to enable it only for a some days and then turn it off.

You may have a long living feature you know it performs bad on highly volume days like a Black Friday campaign: you may use this toggle to turn it off in those moments.

### Permission toggles

They are long lived flags you use to enable some feature only to particular users like **superadmins** or paid users with a PRO plan etc.

Each request will hit this toggle which is usually very dynamic by its nature.

## Increased complexity

Feature Flags have a tendency to multiply rapidly, particularly when first introduced. They are useful and cheap to create and so often a lot are created. However toggles do come with a carrying cost.

> They require you to introduce new abstractions or conditional logic into your code. They also introduce a significant testing burden.

In order to keep the number of feature flags manageable a team must be proactive in removing feature flags that are no longer needed.
Some examples:

- add a toggle removal task onto the team's backlog whenever a Release Toggle is first introduced
- put "expiration dates" on their toggles.
- set a limit on the number of feature flags a system is allowed to have at any one time. Once that limit is reached if someone wants to add a new toggle they will first need to do the work to remove an existing flag.

## Implementation techniques

It is rather easy to introduce a feature toggle in the code by simply adding an **if statement**.
While it may be good for very easy toggles, as soon as you need to check the toggle in different part of the code you are spreading the Toggle decisioning logic in different parts.

### Keep decision points separated from decision logic

```javascript
const features = fetchFeatureTogglesFromSomewhere();

function calculateScoring(guid){
    const transactions = getTransactions(guid);
    if (features.isEnabled("scoring-v2")) { 
      return scoringV2(transactions);
    }else{
      return scoring(transactions);
    }
  }
```

While this looks like a reasonable approach, it's very brittle.
The decision on whether to use the new alghorithm is wired directly in the feature - using a magic string _"scoring-v2"_, no less.

Consider the **scoring-v2** is returning _more_ date then V1 and I need to treat those data if present. I will ahve another place in my code where I have to repeat my if.

What happens if we'd like to turn it on only on Invidual Companies?
What happens if we want to enable it only on onboarding started in April?

So we can refactor using:

```javascript
const features = centralizedFeatureRepository();

function calculateScoring(guid){
    const transactions = getTransactions(guid);
    if (features.isScoringV2Enabled(companyData)) { 
      return scoringV2(transactions);
    }else{
      return scoring(transactions);
    }
  }
  
// ---

export const centralizedFeatureRepository = () => {
    const features = fetchFeatureTogglesFromDB();
    
    return {
        isScoringEnabled: (companyData) => {
            return features.isEnabled("scoring-v2") && companyData.form === 'DI';
       } 
   };
    
};

```

So I can re-use the centralized repository in different part of the code, keeping the toogle logic in one place.

### Inversion of decision

If I don't want my code to have any if expression at all, I can use the power of composition and inversion to achieve the same result.

```javascript
function calculateScoring(guid, scoringAlgorithm){
   const transactions = getTransactions(guid);
   return scoringAlgorithm(transactions);
}
  
// ---

const scoringToBeUsed = centralizedFeaturesRepository.isScoringV2Enabled? scoringV2 : scoring;
calculateScoring(guid, scoringToBeUsed);

```

## Best practices

### Expose current feature toggle configuration

With a static file, a dashboard or a metadata endpoint like Spring Actuator.

### Prefer static configuration

Use code-committed toggles configuration so you can track easily all the changes and know the current status.
Do not redeploy the whole app, just the kubernetes deployement

### Try Azure App Configuration ???
https://docs.microsoft.com/en-us/azure/azure-app-configuration/overview
