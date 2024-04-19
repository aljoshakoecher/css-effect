# CSSE - Formal Definition of Effects in the Context of Capabilities, Skills and Services

## Overview
The CSSE ontology is a lightweight ontology module that formally defines the term *effect* in the context of manufacturing. An effect is defined as the delta in properties between two states - one source state before and one target state after appliying some manufacturing operation. 
On the one hand, there is typically a required effect, which is defined, e.g., by a product designer, a manufacturing planner or resource operator. On the other hand, effects are achieved by capabilities provided by machines.

States can be defined using properties of four different aspects:
* Product Properties: Properties defining certain characteristics of a product to be manufactured or transported (e.g., product weight)
* Process Properties: Properties of the underlying process that carries out processing of products (e.g., cycle time)
* Resource Properties: Properties inherent to resources (e.g., a robot's maximum payload)
* Environment Properties: Properties that cannot be attributed to any of the previous aspects (e.g., ambient temperature)

Following this distinction of properties, there are four types of effects: 
* A ProductEffect is an effect in relation to product properties.
* A ProcesstEffect is an effect in relation to process properties.
* A ResourceEffect is an effect in relation to resource properties.
* An EnvironmentEffect is an effect in relation to environment properties.
Depending on a stakeholder's view, a certain effect may be the only effect to consider while others are considered side effects.

With this ontology, both the effects achieved by capabilities and required effects can be modeled as OWL class expressions. To find out whether a certain capability's effect is suitable to achieve a required effect, the two effects can simply be intersected. If this intersection is empty, the two effects are not compatible. This is explain in more detail in the [usage section below](#usage)

In order to match a capability's effect with a required one in relation to a certain view, a sub requirement needs to be defined that only specifies the properties of the given view. While a required effect may match a capability's effect in one view, an overall match is only achieved if all four sub effects match.

This module is intended as an extension of the [CSS ontology](https://github.com/CaSkade-Automation/CSS).

## Examples

## Usage
### 1. **Import the CSSE ontology**
In order to use this ontology to model effects in your own ontology, you must first import the CSSE ontology using its W3ID IRI `http://www.w3id.org/plattfom-i40/css-effect`. This unversioned IRI always redirects to the latest version of the ontology. You should always use the versioned IRI to prevent unwanted changes to your ontology. An example import in Turtle syntaxt looks something like this:
``` Turtle
<http://www.example.com/myOntology> rdf:type owl:Ontology ;
	owl:imports <http://www.w3id.org/plattform-i40/csse/{version}>> ;
```
Make sure to replace `{version}` with a version of your choice (see all versions [here](https://github.com/aljoshakoecher/css-effect/releases))

### 2. **Define your properties**
The CSSE ontology defines base data properties for the aforementioned types of properties (product, process, resource, environment). Add your properties as subclasses to these properties. As an example, let's consider a property `ex:hasHoleDepth`, which is defined as a subproperty of `CSSE:hasProductProperty`:
```turtle
ex:hasHoleDepth ⊏ CSSE:hasProductProperty
```

### 3. **Define an effect achieved by a capability**
Effects are modeled as OWL class expressions using an initial and target state which may both be specified by properties. Define your requirement by specifying the allowed propety values in the source and target state. Consider the following example, which shows a class expression using the previously defined property for a simplified drilling capability:

```Turtle
ex:DrillingEffect ≡ CSSE:State
 and (CSSE:hasTransitionTo exactly 1 
	(CSSE:State and (ex:hasHoleDepth only xsd:integer[<= 100])
 				and (ex:hasHoleDepth exactly 1 rdfs:Literal))
	)
 and (CSSE:hasTransitionTo exactly 1 CSSE:State)
```

This effect is defined as a source state (first `CSSE:State`) without any restrictions. This state has a transition to exactly one state, the target state. The target state allows all hole depths <=100. Note that the two additional expressiosn `ex:hasHoleDepth exactly 1 rdfs:Literal` and `CSSE:hasTransitionTo exactly 1 CSSE:State` are needed to restrict the open world assumption underlying OWL in order to get valid matches.

### 4. **Define a required effect**
Required effects are modeled in exactly the same way as effects of capabilities. Consider the following required effect:

```Turtle
ex:RequiredEffect ≡ CSSE:State and (
		(ex:hasWidth value 0) and 
		(ex:hasWidth exactly 1 rdfs:Literal)
		)
	and (
		CSSE:hasTransitionTo exactly 1 (
			CSSE:State
			and (ex:hasHoleDepth value 120)
			and (ex:hasHoleDepth exactly 1 rdfs:Literal)
		)
	)
	and (CSSE:hasTransitionTo exactly 1 CSSE:State)
```

This effect specifies a property `ex:hasWidth` on the source state and requires the `ex:hasHoleDepth` property to be 120 on the target state.

### 5. **Match capability effects with requirements**
In order to prove whether a capability's effect can fulfill a given required effect, the two effects need to be intersected and an OWL reasoner like Pellet can be used to check for consistency. In our example, we can define a match as `ex:Match ≡ ex:DrillingEffect ⊓ ex:RequiredEffect`. With the effect definitions given in 3 and 4, this intersection is empty. `ex:Match` will be inferred to be equal to `owl:Nothing`. This is because the required value of 120 for `ex:hasHoleDepth` in the target state of `ex:RequiredEffect` can not be achieved by `ex:DrillingEffect`.

If you want to trigger an inconsistency and let the solver generate an explanation, you can manually assign an individual to the class `ex:Match` so that it cannot be equal to `owl:Nothing` anymore. 

To try out a valid match, simply set the value for `ex:hasHoleDepth` to something <= 100 and start the reasoner again. You should see that the class `ex:Match` is now a not-empty class.




