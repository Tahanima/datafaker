# Custom providers

Since version 1.2.0 of Datafaker it's possible create your own provider of data.

A [full example](https://github.com/datafaker-net/datafaker/blob/main/src/test/java/net/datafaker/providers/base/CustomFakerTest.java) can be found in the source code.


## Custom hardcoded provider

To create a custom provider of data, you'll need to do the following steps:

* Create custom provider of data
* Create your own custom faker which extends `Faker` and register custom provider

In code, this would look like the following:

### Hardcoded provider

Create a custom provider of data:

=== "Java"

    ``` java
    public static class Insect extends AbstractProvider<BaseProviders> {
        private static final WeightedRandomSelector selector = new WeightedRandomSelector(new Random());

        private static final String[] INSECT_NAMES = { "Ant", "Beetle", "Butterfly", "Wasp" };
        private static final List<Map<String, Object>> WEIGHTED_INSECTS = List.of(
            Map.of("value", "Driver ant", "weight", 6.0),
            Map.of("value", "Fire ant", "weight", 3.0),
            Map.of("value", "Harvester ant", "weight", 1.0)
        );

        public Insect(BaseProviders faker) {
            super(faker);
        }

        public String nextInsectName() {
            return INSECT_NAMES[faker.random().nextInt(INSECT_NAMES.length)];
        }

        public String weightedInsectName() {
            return selector.select(WEIGHTED_INSECTS);
        }
    }
    ```
 
### Register provider

Create your own custom faker, which extends `Faker`, and register the custom provider:

=== "Java"

    ``` java
    public static class MyCustomFaker extends Faker {
        public Insect insect() {
            return getProvider(Insect.class, Insect::new, this);
        }
    }
    ```

### Usage

To use the custom faker, you can do the following:

=== "Java"

    ``` java
    MyCustomFaker myFaker = new MyCustomFaker();
    System.out.println(myFaker.insect().nextInsectName());
    ```

This will print something like the following:

```
Wasp
```

**Usafe of weigted random selector is in the POC stage and is currently available only for custom hardcoded providers.**

To use a random selector based on weights, you can do the following:

=== "Java"

    ``` java
    MyCustomFaker myFaker = new MyCustomFaker();
    System.out.println(myFaker.insect().weightedInsectName());
    ```

This will return a random insect name but based on the provided weights
```
Driver ant
```

## Custom provider using Yaml file

In case you have a large set of data to load, it might be better to use a Yaml file.

To create a custom provider of data fom a file, you'll need to do the following steps:

* Create a custom provider of data
* Create your own custom faker which extends `Faker` and register custom provider

### Yaml provider

First, create the custom provider which loads the data from a file:

=== "Java"

    ``` java
    public class InsectFromFile extends AbstractProvider<BaseProviders> {
        private static final String KEY = "insectsfromfile";
        
        public InsectFromFile(BaseProviders faker) {
            super(faker);
            faker.addPath(Locale.ENGLISH, Paths.get("src/test/ants.yml"));
            faker.addPath(Locale.ENGLISH, Paths.get("src/test/bees.yml"));
        }

        public String ant() {
            return resolve(KEY + ".ants");
        }

        public String bee() {
            return resolve(KEY + ".bees");
        }
    }
    ```

The `ants.yml` would look like the following:

```yaml
en:
  faker:
    insectsfromfile:
      ants: ['Driver ant', 'Fire ant', 'Harvester ant', 'Honey ant', 'Leafcutter ant', 'Sahara desert ant']
```

And if you want to use multiple YAML files, the `bees.yml` would look like this:

```yaml
en:
  faker:
    insectsfromfile:
      bees: ['Bumblebee', 'Euglossine bee', 'Honeybee', 'Carpenter bee', 'Leaf-cutter bee', 'Mining bee']
```


### Register provider

Registering the provider would happen like this:

=== "Java"

    ``` java
    public static class MyCustomFaker extends Faker {
        public InsectFromFile insectFromFile() {
            return getProvider(InsectFromFile.class, InsectFromFile::new, this);
        }
    }
    ```

### Usage

To use the custom faker, you can do the following:

=== "Java"

    ``` java
    MyCustomFaker myFaker = new MyCustomFaker();
    System.out.println(myFaker.insectFromFile().ant());
    ```

This will print something like the following:

```
Honey ant
```
