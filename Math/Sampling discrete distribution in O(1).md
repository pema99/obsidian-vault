# Setup - O(n)
- Make bins for each discrete outcome, with the bin containing the probability of that element
- Calculate the average probability of all outcomes
- For each bin with a probability over the mean, redistribute the excess probability to bins with probability less than the mean
	- It is always possible to do this such that each bin contains at most 2 outcomes
	- Can be done via "robin-hood" method, take from the bin with most probability, give to the bin with least
- Each bin now contains 2 outcomes at most, each with their own probability
![[Pasted image 20230917172903.png]]
![[Pasted image 20230917172908.png]]
# Sampling - O(1)
- Generate 2 uniform random numbers in `[0; 1]`
- Use the first number to select which bin to look into
- Use the second number to select which of the 2 (at most) outcomes in the bin to choose

This is also known as the "Alias Method" or "Squaring off the histogram"

Based on https://stats.stackexchange.com/questions/67911/how-to-sample-from-a-discrete-distribution/68041#68041

Example implementation https://github.com/pema99/rust-path-tracer/blob/master/src/light_pick.rs#L24

#math #probability