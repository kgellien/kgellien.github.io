# 2017-12-28-PropertyBasedTesting

## Property based testing

[Testing Web Services
with QuickCheck](https://www.infoq.com/presentations/test-web-service-quickcheck?utm_source=presentations_about_fp&utm_medium=link&utm_campaign=fp)

Thomas Arts discusses how to test web services with QuickCheck.

A very nice and engaging  conference talk about property absed testing with the Erlang/Elixir version of QuickCheck.
He begins with a short introduction into the basic ideas of property based testing.

Then follows some advice on how to create properties out of specifications.

In the last part of his talk he talks about what to to when there are no proper specifications.
If some JSON schemas are present, these might be used to formulate properties to test.
He uses JSON schemas to specify data and then use this to generate arbitrary data.

But arbitrary data can be too "arbitrary". Many properties can not be formulated inside of a JSON schema. So there quickly is demand for more precise specifications.
Then QuickCheck can be used as a Domain Specific Language for defining random data.
