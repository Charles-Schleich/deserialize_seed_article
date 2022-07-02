An intermediate guide to maunally implementing Serde's deserializing with state injection:


A couple of days ago i ran into the problem where i was deserializing a block of JSON, i had a problem :
I needed some extra information to be added to the struct which serde's deserialzation was targeting, which was not present inside the JSON.
One option i had was to:
I would either need to deserialize the JSON into struct A then convert to Struct B
we could do better. 

Struct A with an Optional<TypeX>.
Okay heading in the right direction, but it still meant that we needed to effectively process the data twice, once while deserializing to struct A, and a second time when we were injecting TypeX. This also Lead to a scenario where everywhere later in our code we would have to handle the fact that TypeX was wrapped in an optional, which seems like bad design because it always should be a Some(variant).
We could unwrap, but the longer you write production rust code, the more that the word unwrap sounds like a swear word.

So i sought out to do better than both of these options above and im going to write up today how to solve this exact problem with some intermediate rust serde examples as well as explain my learnings of manually implementing Deserialize/

Get in, buckle up, lets party.

To sum up what we are trying to accomplish in 2 code blocks

```json
{   "outerstr":"String",
    "outeru8": 123,
    "nested1": {
        "innerstr":"value",
        "innervec": [
            {
                "vecfloat1":"0.10",
                "vecint1":""
            },
            {
                "vecfloat1":"0.20",
                "vecint1":""
            },
            {
                "vecfloat1":"0.30",
                "vecint1":""
            },
            {
                "vecfloat1":"0.40",
                "vecint1":""
            },
        ]
    }

}
```

And we want to deserilaize this into the Rust type

```Rust
    struct MyStruct {
        
    }

    enum MyEnum {

    }
```


You see the problem ? 
`MyStruct` is expecting the presence of an `MyEnum` which is not present in the json we are trying to deserialize.

Looks like we are going to have to figure out how to solve this problem,