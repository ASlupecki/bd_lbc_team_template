# 4Better Design Document

## Dog Matching Service Design

## 1. Problem Statement

When potential dog owners are searching for a new dog, they can find themselves mismatched.
Whether the dog they get has too much energy, is too hard to train, etc.  Many pet sites
and city shelters only post photos of the animal and sometimes, loose descriptions.  It's difficult
to determine if the dog will adequately fit into your lifestyle.

This design document describes a 'matching' service, that will allow dog sellers to easily post a dog's profile,
containing a basic set of easily filterable traits.  Dog shoppers can then search dogs
by desirable traits for their particular needs.

## 2. Top Questions to Resolve in Review

1.   Do we want to attempt a filter that checks a customer's geographic proximity to a dog?  Or is that to ambitious for this scope?
2.   Do we want to include physical traits as a search criteria (ie: size, color, ear set, muzzle type) or just let the photos show that?
3.   For the scope of this project, how many filterable traits do we want to limit ourselves to? (we can add more later, but this project can easily get too big)

## 3. Use Cases

U1. As a pet owner, I want to find results that match my criteria when I sort and filter

U2. As a pet owner, I want to be able to contact a seller when i find a dog I'm 
interested in

U3. As a pet owner, or a dog seller, I want to be able to understand the 
criteria used by the service, and how it applies to me

U4. As a dog seller, I want to be able to easily post available dogs

U5. As a dog seller, I want to be easily contactable by interested owners

U6. As a dog seller, I want to be able to easily update information on 
available dogs

U7. As a dog seller, I want to be able to easily remove dogs from the service
when they become unavailable

## 4. Project Scope

### 4.1. In Scope

1. Dog sellers can easily create, fill out, and post a dog for sale with attributes related to temperament and behavior
2. Dog sellers can easily update, or remove, the dog's post 
3. Dog owners can easily sort and filter available dogs for sale by attributes related to temperament and behavior
4. Dog owners can find contact information for the dog seller of a dog that they're interested in

### 4.2. Out of Scope

1. Criteria focused on physical attributes of the dogs, beyond basics (size/coat type)
2. Facilitating communication between a seller and owner beyond providing contact information (ie: messaging service)
3. Filtering by breed or breed mix
4. In the future, I'd like the ability to provide a quiz for customers to take, but that's out of scope for now
5. Verification to prevent false posts/fake users.
6. Any monetary handling/funds exchange: This service is not intended to be Amazon for dogs, its primarily for shelter/rescue type clients
7. Separating 'traits' into individual attributes with unique getters/setter instead of a list

# 5. Proposed Architecture Overview

This iteration of the project will provide the minimum lovable product: including
creating, retrieving, and updating a dog's information for the seller, as well as allowing the
owners/buyers to retrieve a dog's information by trait.

We will use API Gateway and Lambda to create seven endpoints (`GetDog`,
`CreateDog`, `UpdateDog`, `GetDogByTrait`, `GetSellerContact`, `UpdateSeller`, `CreateSeller`)
that will handle the creation, update, and retrieval of a dog's information to satisfy our
requirements.

We will store all dogs in a table in DynamoDB.  They will have a randomly generated ID as an 
individual value partition key, and a sort key of the seller ID.

All sellers will also exist in a table, with a randomly generated ID as a partition key,
and name as the sort key.

Sellers can have many dogs for sale, or zero, but dogs can only have one seller at a time.

Dog Matching Service will also provide a web interface for sellers to manage
their dogs for sale, and for buyers to view available dogs. A main page providing a list 
view of all the dogs available will let sellers view every dog they sell, and create a new listing.
The main page will link off to a unique page per dog to update info or delete the dog from their list.

For Buyers, they will be able to access only the information, not change or update anything, and search by
criteria. In an individual dog's profile, they can access information to contact the seller.

# 6. API

## 6.1. Public Models

```
// SellerModel

String id;
String name;
String contact;
List<DogModel> dogs;
```

```
// DogModel

String id;
String name;
SellerModel seller;
List<String> traits;
```

## 6.2. Get Dog Endpoint

* Accepts `GET` requests to `/dogs/:id`
* Accepts a dog ID and returns the corresponding DogModel.
    * If the given dog ID is not found, will throw a
      `DogNotFoundException`

## 6.3 Create Dog Endpoint

* Accepts `POST` requests to `/sellers`
* Accepts data form a seller to create a new dog with a provided name, provided seller, and a list of traits.
* Returns the new dog, including a unique ID assigned by the Match Service.
* For security concerns, we will validate the provided dog name does not
  contain any invalid characters: `" ' * # $ \`
    * If the dog name contains any of the invalid characters, will throw an
      `InvalidAttributeValueException`. 
![Client sends create new dog form to website create dog page. Website 
 create dog page sends an create new dog request to CreateDogActivity.
 CreateDogActivity saves the new dog to the dogs
 database.  It also automatically updates the dog to the seller's list of 
 dogs](diagrams/CreateDogSequence.png)

## 6.4 Update Dog Endpoint

* Accepts `PUT` requests to `/sellers/dogs/:id`
* Accepts data to update a dog including an updated dog name, updated list of traits, and the dog ID associated with the individual dog. 
* Returns the updated dog.
    * If the dog ID is not found, will throw a `DogNotFoundException`
* For security concerns, we will validate the provided dog name does not
  contain invalid characters: `" ' * # $ \`
    * If the dog name contains invalid characters, will throw an
      `InvalidAttributeValueException`
![Client sends submit dog update form to Website update dog information page. Website
update dog information page sends an update request to UpdateDogActivity.
UpdateDogActivity saves updates to the dogs
database.](diagrams/UpdateDogSequence.png)

## 6.5 Get Dog By Trait Endpoint

* Accepts `GET` requests to `/dogs/:traits`
* Accepts a valid trait from an ENUM list and returns a list of corresponding DogModels that include this trait.
    * If no dog is not found with the desired trait, will throw a
      `DogNotFoundException`
    * If the provided trait is not a valid one from the ENUM list, will throw a
      `InvalidTraitException`

## 6.6 Get Seller Contact Endpoint

* Accepts `GET` requests to `/seller/:id` and `/dogs/:id`
* Accepts a seller ID and returns the corresponding SellerModel's contact information.
    * If the given seller ID is not found, will throw a
      `SellerNotFoundException`
![Client requests seller contact from Website dog information page. Website
dog information page sends a get seller contact request to GetSellerContactActivity.
GetSellerContactActivity retrieves the seller from the dogs database. It then requests the
seller contact of that seller from the seller database and returns it to the 
client](diagrams/GetSellersContactSequence.png)

## 6.7 Update Seller Endpoint
* Accepts `PUT` requests to `/sellers/:id`
* Accepts data to update a seller including an updated name, and updated contacts.
* Polymorphism should allow this to change without affecting the dogs profiles
* Returns the updated seller.
* For security concerns, we will validate the provided seller name does not
  contain invalid characters: `" ' * # $ \`
    * If the seller name contains invalid characters, will throw an
      `InvalidAttributeValueException`

## 6.3 Create Seller Endpoint

* Accepts `POST` requests to `/sellers`
* Only used upon creation of a new account.
* Accepts data form a seller client to create a new seller account with a provided name, and contact.
* Returns the new seller, including a unique ID assigned by the Match Service.
* For security concerns, we will validate the provided seller name does not
  contain any invalid characters: `" ' * # $ \`
    * If the dog name contains any of the invalid characters, will throw an
      `InvalidAttributeValueException`.


# 7. Tables

### 7.1. `dogs`

```
id // partition key, string
name // string
seller // sort key, string
traitList // list
```

### 7.2. `sellers`

```
id // partition key, string
name // string
contact // string
dogList // list
```

# 8. Pages

![The Home page offers the choice between selling a dog or looking for a dog.
It displays a simple explanantion of the service, followed by the coice between two links.
The first link will lead to a log in form, the second to the view dogs page with all available
dogs on the service.  This differentiates between the two types of clients](images/4Better/HomePage.png)

![The View Dogs page has a selection bar at the top that allows users to select a trait.  This 
will eliminate dogs from the selection that don't include this trait in their 'traits' list.
The rest of the page shows a list of dogs.  Selecting any dog will navigate to its dog page.
Seller clients will have access to a similar page that only shows THEIR dogs](images/4Better/ViewDogs.jpg)

![The dog page displays the dog's name at the top, along with a picture and a list of traits
that apply to the dog.  Sellers viewing their own dogs will see an option to update dog and
delete dog (not shown).  Buyers will see a link to the contact seller form to request
the contact information for the seller if they are interested in the dog](images/4Better/DogPage.png)

![The seller interface includes the name of the seller at the top, and three options of buttons.
View Dogs button takes seller to a list of their currently available dogs, create new dog button
takes them to the create dog form, and the update profile takes them to the update seller profile
form (not shown)](images/4Better/SellerInterface.png)

![The Create Dog page is a form that requires a name and at least one trait to be selected.
There is a 'create' button at the bottom.](images/4Better/CreateDog.png)

## 8.1 Pages Not Shown

* The Update Dog page is virtually identical to the Create dog page, 
except the button and header say "update"
* The Create Seller and Update Seller form are simple identical forms that take in a name
and contact information like email, phone number, address as required fields.  Like the 
create and update dog pages, the only difference is the words in the header and button.
* The Delete Dog option redirects to an "are you sure" type page, before a "yes" or "no"
button commits to the action.
