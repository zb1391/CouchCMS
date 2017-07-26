# Goal
To figure out how Services / Add Ons will be communicated between Open Bravo and The Red Door. We are trying to completely change the way services are selected. In the current system, the number of combinations a user can choose from is bound by the sku itself. We are hoping to move to a place where the user can customize the service with a combination of add ons.

First a user chooses a Service:

![](https://s18.postimg.org/7ob5qqfgp/step1.png)

Then a user chooses to customize:

![](https://s22.postimg.org/c41mbd2z5/step2.png)

On the Nails Page, however, these three tiers are separated by a sub navigation for you to choose between Manicures, Pedicures, and Gels & Acrylics.

There are certain rules around how things can be customized:
* Each sku has a maximum number of add-ons. (For example, the mini you can choose a maximum of 3)
* Certain combinations cannot be done (For example, the mini you can only have one led technology)
* Certain Add-Ons are only present based on the users position (For example, CACI in NYC)

## definitions
* **add-on**: a supplement to a service that does not add time
* **upgrade**: a supplement to a service that does add time
* **category**: Face, Body, Nails, Wax, etc.
* **subcategory**: For nails -> Manicure, Pedicure, Gels, etc

## Problems / Questions
* Open Bravo currently does not support upgrades. They are treated as separate skus
* Open Bravo currently does not support Sub Categories.
* Open Bravo has no way of knowing which combinations of add-ons are acceptable for each sku
* How do I group each add-on into the appropriate column. (How do I know that LED is a technology?)
* How is the data shared between Open Bravo and The Red Door
* Currently skus are associated to technicians. Are ther add-ons that a training specific? Do we need to handle a case where someone who wants a specific add on accidentally books someone who can do the sku but not that add on? (Is this even a problem)
* How does any changes / solutions provided affect the internal booking tools used by Front Desk and the call center?
## Solution 1 - Keep as much the same as possible
Get rid of the idea of add-ons. Each service is and combination is its own sku. The name of the sku includes the add-ons with a special delimiter, sorted in alphabetical order. For example:
```
[
  {
    "name": "Mini",
    "category": "Face",
  },
  {
    "name": "Mini+LED",
    "category": "Face",
  },
  {
    "name": "Mini+OxygenBlast",
    "category": "Face"
  },
  {
    "name": "Mini+VitaminCMask",
    "category": "Face"
  }
  ...
  {
    "name": "Mini+LED+VitaminCMask",
    "category": "Face",
  }
  ...
]
```
### Does it solve the problems?

**Open Bravo currently does not support upgrades. They are treated as separate skus**

```this is solved by having each combination be its own sku with its own duration.```

**Open Bravo currently does not support Sub Categories.**

```this is not resolved. I think a sub-category field is going to be required no matter what solution is chosen```

**Open Bravo has no way of knowing which combinations of add-ons are acceptable for each sku**

```
this is handled by a client side validation. For example, lets say a customer wants a Mini with Oxyegen Blast and then LED. Every checkbox 

let service = "Mini"
let add_ons = ["OxygenBlast","LED"]
let sorted = ["LED", "OxygenBlast"]
service = service + sorted.join("+") // --> Mini+LED+OxygenBlast

I can then validate if this service exists or not
I can also recommend upgrades. For example, I can then look for:
Essential+LED+OxygenBlast and Escape+LED+OxygenBlast
If it exists I can recommend the user to upgrade
```

**How is the data shared between Open Bravo and The Red Door**

```The updater program would not need to significantly change. I am just grabbing each one at a time and storing it on my end. We would need to add logic to import the sub-category as well, unless we think of a way around it.```

**Currently skus are associated to technicians. Are ther add-ons that a training specific? Do we need to handle a case where someone who wants a specific add on accidentally books someone who can do the sku but not that add on? (Is this even a problem)**

```This is not a problem since each service is a sku now.```

**How does any changes / solutions provided affect the internal booking tools used by Front Desk and the call center?**

```Everything should stay exactly the same.```

### Problems
This is going to create an insane amount of skus. In Face, for just the Mini, it would create 79 skus. That is excluding any secret menu skus that might exist. The number increases exponentially when you add the Essential and Escape that offer multiple choices in each add on category.

## Solution 2
What if we apply Solution 1 so that Upgrades and Secret Menu Items are separate skus? This would tremendously decrease the total number of skus created.

To me the problem then is how to I validate the combinations? How do I know that You cannot get a Mini+LED+OxygenBlast? We would need to store these rules somewhere. We could store them on my side of things, but this affects the Open Bravo internal tools. How would a front desk operator know that this combination is not permitted if the internal tool has no way of validating?

This seems to be the biggest issue with the entire process. Making sure it is viable for internal tools as well...
