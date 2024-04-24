# CSS-Effect - Formal Definition of Effects in the Context of Capabilities, Skills and Services

## Overview
The CSS-Effect ontology is a lightweight ontology module that formally defines the term *effect* in the context of manufacturing. An effect is defined as the delta in properties that occurs during a transition between two states - one source state before and one target state after applying some manufacturing operation. 
On the one hand, there is typically a required effect, which is defined, e.g., by a product designer, a manufacturing planner or resource operator. On the other hand, effects are achieved by capabilities provided by machines.

States can be defined using properties of four different aspects:
* Product Properties: Properties defining certain characteristics of a product to be manufactured or transported (e.g., product weight)
* Process Properties: Properties of the underlying process that carries out processing of products (e.g., cycle time)
* Resource Properties: Properties inherent to resources (e.g., a robot's maximum payload)
* Environment Properties: Properties that cannot be attributed to any of the previous aspects (e.g., ambient temperature)

These properties can also be used to model properties of the transition, i.e., invariants which must not only hold at the source or target state, but throughout the whole transition.

Following this distinction of properties, there are four types of effects: 
* A ProductEffect is an effect in relation to product properties.
* A ProcesstEffect is an effect in relation to process properties.
* A ResourceEffect is an effect in relation to resource properties.
* An EnvironmentEffect is an effect in relation to environment properties.
Depending on a stakeholder's view, a certain effect may be the only effect to consider while others are considered side effects.

The CSS-Effect is defined in the following namespace and the prefix CSSE is used throughout the documentation:
```
@prefix CSSE: <http://www.w3id.org/plattfom-i40/css-effect#> .
```
With this ontology, both the effects achieved by capabilities and required effects can be modeled as OWL class expressions. To find out whether a certain capability's effect is suitable to achieve a required effect, the two effects can simply be intersected. If this intersection is empty, the two effects are not compatible. This is explain in more detail in the [usage section below](#usage)

In order to match a capability's effect with a required one in relation to a certain view, a sub requirement needs to be defined that only specifies the properties of the given view. While a required effect may match a capability's effect in one view, an overall match is only achieved if all four sub effects match.

This module is intended as an extension of the [CSS ontology](https://github.com/CaSkade-Automation/CSS). CSSE contains an object property `achievesEffect` to link a `CSSE:Effect` with a `CSS:Capability`.

## Usage
### 1. **Import the CSS-Effect ontology**
In order to use this ontology to model effects in your own ontology, you must first import the CSS-Effect ontology using its W3ID IRI `http://www.w3id.org/plattfom-i40/css-effect`. This unversioned IRI always redirects to the latest version of the ontology. You should always use the versioned IRI to prevent unwanted changes to your ontology. An example import in Turtle syntaxt looks something like this:
``` Turtle
<http://www.example.com/myOntology> rdf:type owl:Ontology ;
	owl:imports <http://www.w3id.org/plattform-i40/css-effect/{version}>> ;
```
Make sure to replace `{version}` with a version of your choice (see all versions [here](https://github.com/aljoshakoecher/css-effect/releases))

### 2. **Define your properties**
The CSS-Effect ontology defines base data properties for the aforementioned types of properties (product, process, resource, environment). Add your properties as subclasses to these properties. As an example, let's consider a property `ex:hasPocketDepth`, which is defined as a subproperty of `CSSE:hasProductProperty`, and a property `ex:hasMillTemperature`, which is a subproperty of `CSSE:hasResourceProperty`:
```turtle
ex:hasPocketDepth ⊏ CSSE:hasProductProperty
ex:hasMillTemperature ⊏ CSSE:hasResourceProperty
```

### 3. **Define an effect achieved by a capability**
Effects are modeled as OWL class expressions using an initial and target state which may both be specified using properties. Define your requirement by specifying the allowed propety values in the source and target state. Consider the following example, which shows a class expression using the previously defined property for a simplified milling capability:

```Turtle
ex:MillingEffect ≡ CSSE:Effect and
(ex:hasMillTemperature max 300) and
(ex:hasMillTemperature exactly 1 rdfs:Literal) and
(CSSE:hasSourceState some 
    (CSSE:State
     and ((ex:hasPocketDepth value 0)
     and (ex:hasPocketDepth exactly 1 rdfs:Literal))))
 and (CSSE:hasTargetState some 
    (CSSE:State
     and ((ex:hasPocketDepth only xsd:integer[<= 200])
     and (ex:hasPocketDepth exactly 1 rdfs:Literal))))
 and (CSSE:hasSourceState exactly 1 CSSE:State)
 and (CSSE:hasTargetState exactly 1 CSSE:State)
```
`ex:MillingEffect` defines an invariant on `ex:hasMillTemperature`, which is guaranteed to be less than 300. It then defines constraints on the source and target state.
In the source state, the pocket depth must be exactly 0. The target state allows all pocket depths <=200. Note that all additional expressions (i.e., `hasMillTemperature max 1 rdfs:Literal`, `ex:hasPocketDepth exactly 1 rdfs:Literal`, `CSSE:hasSourceState exactly 1 CSSE:State` and `CSSE:hasTargetState exactly 1 CSSE:State` are needed to restrict the open world assumption underlying OWL in order to get valid matches.

### 4. **Define a required effect**
Required effects are modeled in exactly the same way as effects of capabilities. Consider the following required effect:

```Turtle
ex:RequiredEffect ≡ CSSE:Effect and
(ex:hasMillTemperature max 300) and
(ex:hasMillTemperature exactly 1 rdfs:Literal) and
(CSSE:hasSourceState some 
    (CSSE:State
     and ((ex:hasPocketDepth value 0)
     and (ex:hasPocketDepth exactly 1 rdfs:Literal))))
 and (CSSE:hasTargetState some 
    (CSSE:State
     and ((ex:hasPocketDepth value 300)
     and (ex:hasPocketDepth exactly 1 rdfs:Literal))))
 and (CSSE:hasSourceState exactly 1 CSSE:State)
 and (CSSE:hasTargetState exactly 1 CSSE:State)
```

This effect specifies `ex:hasPocketDepth` on the source state to be 0 and requires the `ex:hasPocketDepth` property to be 300 on the target state.

### 5. **Match capability effects with requirements**
In order to prove whether a capability's effect can fulfill a given required effect, the two effects need to be intersected and an OWL reasoner like Pellet can be used to check for consistency. In our example, we can define a match as `ex:Match ≡ ex:MillingEffect ⊓ ex:RequiredEffect`. With the effect definitions given in 3 and 4, this intersection is empty. `ex:Match` will be inferred to be equal to `owl:Nothing`. This is because the required value of 300 for `ex:hasPocketDepth` in the target state of `ex:RequiredEffect` can not be achieved by `ex:MillingEffect`.

If you want to trigger an inconsistency and let the solver generate an explanation, you can manually assign an individual to the class `ex:Match` so that the inference of `owl:Nothing` causes an inconsistency. 

To try out a valid match, simply set the value for `ex:hasPocketDepth` to something <= 200 and start the reasoner again. You should see that the class `ex:Match` is now a not-empty class.

## Examples
You can find more examples along with explanations in the examples directory.


