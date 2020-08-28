# Flake for QE - Short summary from project hand off

**Existing code base:** 

[https://gitlab.com/opendatahub/ai-library/-/tree/master/flakes_train](https://gitlab.com/opendatahub/ai-library/-/tree/master/flakes_train) 

[https://gitlab.com/opendatahub/ai-library/tree/master/flakes_predict](https://gitlab.com/opendatahub/ai-library/tree/master/flakes_predict)

**How Does it work?:**

**Data:**

Format:

The data that is input in a jsonl format. Each record in the input is a separate line. See test-example.jsonl.gz The following fields are required:

*   "status": String value of `"failure"`, `"success"`, `"error"` or `"skip"`
*   "test": Name of the test
*   "log": The full textual log of the test

Generating:

This merged PR indicates an automated means of producing a labeled training dataset:

[https://github.com/cockpit-project/cockpit/pull/7349](https://github.com/cockpit-project/cockpit/pull/7349)

**ML Process:**

Given a JSONL file in the form outlined above. We do the following:  

Use sklearn CountVectorizer to convert the log message into a 1xN vector where each position represents a word in the vocabulary and each value is the number of times that word occurred in the log message.  
![alt_text](assets/images/image0.png "image_tooltip")

[https://github.com/cockpit-project/cockpituous/blob/208a7be38ab89c14b152127ec679d3627512a267/learn/extractor.py#L112](https://github.com/cockpit-project/cockpituous/blob/208a7be38ab89c14b152127ec679d3627512a267/learn/extractor.py#L112) 

Then we use DBSCAN with a normalized compression distance (ncd) to cluster our training dataset.

![alt_text](assets/images/image1.png "image_tooltip")

[https://github.com/cockpit-project/cockpituous/blob/208a7be38ab89c14b152127ec679d3627512a267/learn/cluster.py#L204-L216](https://github.com/cockpit-project/cockpituous/blob/208a7be38ab89c14b152127ec679d3627512a267/learn/cluster.py#L204-L216) 

K-nearest neighbor is then used for prediction to determine which class a new test log belongs to.

![alt_text](assets/images/image2.png "image_tooltip")

[https://github.com/cockpit-project/cockpituous/blob/208a7be38ab89c14b152127ec679d3627512a267/learn/cluster.py#L242](https://github.com/cockpit-project/cockpituous/blob/208a7be38ab89c14b152127ec679d3627512a267/learn/cluster.py#L242) 

**How is a Flake determined?** It seems to me that it has something to do with a cluster's merge percent. If a point is labeled as failed, you use KNN to figure out which cluster it belongs in, and if it’s matched with a cluster that has a merge percent higher than the flake threshold set in cluster.py, then it is labeled as a flaWas hard to find, butke.   

![alt_text](assets/images/image3.png "image_tooltip")

[https://gitlab.com/opendatahub/ai-library/-/blob/master/flakes_train/bots/learn/cluster.py#L112](https://gitlab.com/opendatahub/ai-library/-/blob/master/flakes_train/bots/learn/cluster.py#L112)

[https://gitlab.com/opendatahub/ai-library/-/blob/master/flakes_predict/bots/tests-policy#L177](https://gitlab.com/opendatahub/ai-library/-/blob/master/flakes_predict/bots/tests-policy#L177)