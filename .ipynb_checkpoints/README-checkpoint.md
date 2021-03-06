## Random Binary Strings

The requirement for this challenge was to discern between human-generated and computer-generated strings of 1s and 0s. A text file, `strings.txt` was provided with 124 lines, each containing a string of 150 1s and 0s. Half of these were computer generated and half were generated by a human without any external randomness aids.

Each line of the file can be thought of as a binomial distribution with 150 sequential trials. The central assumption for this task is that the computer-generated strings will be independent, identically distributed experiments, with probability of success at each trial 0.5. However, the human-generated strings are unlikely to be fully independent. I predict that natural patterns will have been incurred, and identifying these patterns will be key to finding the human-generated entries. 

## Approach

I will be operating at a 95% confidence interval for these experiments. In most cases my null hypothesis is that at each sub-experiment in the sequence is an independent Bernoulli trial with a probability of success of 0.5. Disproving this hypothesis at 95% confidence will be the flag for human input. 

I will be analysing each distribution independently, in order to evaluate whether it might be human-generated. With more time I might also analyse them in relation to one another, as a pertinent fact given is that the samples are split evenly between computer and human-generated data. With more time I could implement a ranking algorithm based on the various probability criteria identified, and split the sorted data down the middle. 

For this implementation I am checking several different probability vectors, and marking the string as human-generated if any test flags them as likely to be. This approach is simple, but has drawbacks because of the small sample size of 150 trials per experiment. Since there are only two outcomes the variance is very high, and this has lead to quite a high false positive rate (~30%). With more time I might have experimented with this approach slightly, perhaps only marking a string as human generated if more than one test fails. This would probably limit false positives, but might also limit true positives. 

## Tests

### Clean Data Test
This is just an initial check to ensure that the string only contains 1s and 0s. If any other characters are included in the sample, it can be assumed that it was human-generated.

### Binomial Test
This just tests for an overall binomial distribution throughout the entire sample with probability 0.5. It is basically ensuring that the spread of 1s and 0s is normally distributed around the mean of 75. Anything outside of two standard deviations of the mean will be flagged as human-generated. Due to the variance of the experiment, this might be likely to catch some false positives. 

### Markov Tests
These tests are working under the assumption that the recent history of inputs will have an impact on the outcome of the following input for a human-generated experiment. Since each computer-generated trial will be independent of any other trial, the conditional probability should always be centred on 0.5. 

This test aims to analyse the patterns between previous inputs and the next outcome. For example, in one human-generated experiment the person might be very likely to input a 1 after a 0. The probability of 0 -> 1 might be 90%, and the probability of 0 -> 0 only 10%. This test will flag such patterns as probably human-generated. It also tests for patterns with a slightly longer history, for example 10 -> 1.

The test builds a transition outcome matrix, which counts all occurrences of each transition. It builds n-grams of the string, and increments the relevant cell at each stage. The columns represent the previous pattern, and rows represent the current outcome. Dividing each value by the sum of its column will get the probability of that transition. If the p-value of that distribution is less than 0.05, it will be flagged as probably human-generated.

### Pattern Matching
These tests are similar to the markov tests, as they are grouping patterns of 1s and 0s together and analysing them. However instead of analysing the transition probability, it checkswhether a particular pattern occurs an unlikely number of times across the entire sample. The assumption here is that a person might be likely to do the same pattern over and over, which might seem random across the entire sample but is unlikely for a computer to do. 

This uses the same logic as above to build the transition matrix, but then flattens it to just get a count of all patterns at size n. Flattening the matrix basically concatenates the column and row values into one pattern. Then dividing by the count of all occurences would get the probability of that pattern. It runs through patterns of different sizes (from 3 to 5), and only flags if the occurrence of that pattern is used more than 3 standard deviations more the mean at size n. 

This uses a 99.7% confidence interval, as patterns do occur regularly in computer-generated experiments. I wanted to be sure that the pattern was very likely to be human-generated before flagging. At a 95% confidence interval it was catching a lot of false positives.

## Results
**Training Data**

False Positive rate: 29.8% (298/1000).

True Positive rate: 96.67% (29/30).

**Test Data**

The tests flagged 55/124 samples as human-generated. I am yet to get results on how accurate these are.

### Gathering Training Data
The computer-generated data was created using the same function used to create the test data. The only difference was that I seeded the RNG to ensure I got the same dataset on each run. 

The human-generated data was gathered from four different sources. Each person was challenged to create a string of 1s and 0s as randomly as possible, without using any external randomness aids. The true positive rate was very high from these sources. I suspect the context of the experiment could have a significant impact on the output of these sources, such as the environment, the input method, the speed at which they input the data and how concentrated they are in creating a string as randomly as possible. 
