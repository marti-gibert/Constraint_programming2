# ESPECIALITZACIÓ DE CIÈNCIES DE DADES: CONSTRAINT PROGRAMMING FOR DATA SCIENCE

First of all let’s break down the problem to understand it correctly:

We have different parameters:

```
given k : int(1..)
```

k is the number of rules to be used in the model that is an integer greater than or equal to 1.

```
given lambda : int(1..)
```

lambda: representing the penalty for misclassifying a sample. Its value is an integer greater than or equal to 1.

```
given samples : matrix indexed by [int(1..noSamples), int(1..noFeatures)] of int(0..1)
```
This matrix contains samples as rows and features as columns. Each element of the matrix is a binary representation of the features.

```
given labels : matrix indexed by [int(1..noSamples)] of int(0..1)
```

Labels contains the binary labels for each sample (0/1).

	
and we have the following variables:
```
find eta : matrix indexed by [int(1..noSamples)] of bool
```
eta is an array to identify whether each sample is correctly identified. So if the first value of eta is 1, means that the first sample have been correctly identified whereas if its value is a 0, it means that i have been incorrectly classified.

```
find rules : matrix indexed by [int(1..k), int(1..noFeatures)] of bool
```

rules matrix represents the rules applied to the model. We have k rules and each rule applies to each feature.

```
minimising objVar

such that
    objVar = sum (flatten(rules)) + sum sample : int(1..noSamples) . lambda * (!eta[sample]),
```

objvar represents the objective function’s value, which we aim to minimize. It contains the total number of active rules (sum (flatten(rules))) and a penalty for misclassified samples (sum sample : int(1..noSamples) . lambda * (!eta[sample]))

```
forAll sample : int(1..noSamples) .
    (labels[sample] = 1) ->
    (
        eta[sample] ->
        (
            forAll rule : int(1..k) .
                exists feature : int(1..noFeatures) .
                    (samples[sample, feature] = 1) /\ rules[rule, feature]
        )
    ),
```
Here, we iterate over each positively labeled sample (labeled as 1). 'eta[sample] ->' implies correct classification based on the current set of rules. We then iterate over 'k' rules to verify if at least one feature in the sample matches a feature in the rule. This constraint ensures that each correctly classified positively labeled sample is matched by at least one rule.

So, now we have to develope contraints for negatively labeled samples. In this case, the constraint needs to ensure that negatively labeled samples are correctly classified as negative by at least one rule.

```
forAll sample : int(1..noSamples) .
    (labels[sample] = 0) ->
    (
        eta[sample] ->
        (
            exists rule : int(1..k) .
                forAll feature : int(1..noFeatures) .
                    (samples[sample, feature] = 0) \/ (!rules[rule, feature])
        )
    )
```
This is achieved by ensuring that for each rule, there exists at least one feature where either the sample does not have the feature (samples[sample, feature] = 0) or the rule does not require the feature (!rules[rule, feature]).


## ANALYSING THE RESULTS WITH THE GIVEN EXAMPLE

Given the parameters specified in the moodel document:

```
language ESSENCE' 1.0

letting k = 2
letting lambda = 30
letting samples =
[
    [1,0,1],
    [0,1,1]
]
letting labels = [0,1]
```
We have 2 samples with features: 

[1,0,1] labelled as 0

[0,1,1] labelled as 1


we want to find 2 rules (k=2) that allow us to classify the samples as it's corresponding labels based on it's features. The solution provided by savile-row is the following one:

```
 language ESSENCE' 1.0
$ Minion SolverNodes: 25
$ Minion SolverTotalTime: 0
$ Minion SolverTimeOut: 0
$ Savile Row TotalTime: 0.098
letting eta be [true, true]
letting objVar be 2
letting rules be [[false, false, true], 
[false, true, false]]
```
By looking at the eta, we can see that both samples have been correctly classified.

We can see that we have obtained 2 rules that allows us to classify all the samples correctly:

rule 1: [false, false, true] indicates that for a sample to be classified as positively by this rule, the third feature must be true (1).
rule 2: [false, true, false] indicates that for a sample to be classified as positively by this rule, the second feature must be true (1).

If we apply these rules in the samples:

sample 1: [1,0,1] matches the rule 1 as the 3rd feature is positive but does not match the rule 2 as the second rule as it implies to have the second feature positive and in this sample is negative.
As the sample does not match all the rules, it is correctly classified as negative.

sample 2: [0,1,1] matches the rule 1 as the 3rd feature is positive and also matches the rule 2 as the second feature is also positive. As the sample matches all the rules, it is correctly classified as positive.


With this simple example, we could use only one rule (k=1) to resolve it, as we can see that we can differenciate the label by only looking at the 2nd feature. If we run it only using one rule the result is the following:

```
language ESSENCE' 1.0
$ Minion SolverNodes: 12
$ Minion SolverTotalTime: 0.000628
$ Minion SolverTimeOut: 0
$ Savile Row TotalTime: 0.114
letting eta be [true, true]
letting objVar be 1
letting rules be [[false, true, false]]
```

We can see that by just looking at the second feature we have been able to correctly identify each sample to its label.


2. include symmetry breaking constraints that accounts with the fact that the order of the rules is irrelevant.

The symmetry breaking constraints help the solver avoid redundant solutions by enforcing a specific order or structure on the rules. In this case, we have to develope a constraint that ensures that the rules are ordered based on their feature activations. By comparing the sum of feature activations for pairs of rules, we establish an order that breaks symmetry.


```
forAll r1 : int(1..k-1) , r2 : int(r1+1..k) .
    sum f1 : int(1..noFeatures) . toInt(rules[r1, f1]) >= sum f2 : int(1..noFeatures) . toInt(rules[r2, f2])
```



TO DO EXPLAIN THIS PART(WORKS CORRECTLY BUT EXPLAIN A LITTLE BIT)


APPLY TO THE TEST FILE PROVIDED BY MATEU (CHANGE THE K AND LOOK WHICH K WORKS BETTER TAKING INTO ACCOUNT THE eta OF .SOLUTIONS FILE CREATED (WHICH K GIVES MORE TRUES))

EXTRA PART: FIND SOME DATASET OF SIMILAR SIZE AND APPLY THE MODEL