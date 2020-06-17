# ng-ui2qp

ng-ui2qp is an angular library which saves UI components state as query parameters in the URL, making it easy to 
serialize UI state into the URL.

## Description
 
The lib gives you a way to synchronizing the state of UI components to the URL as query parameters.
Using the library and the features it provides can improve the user experience in many ways, think in the next use cases:
 
* Accidentally or intentionally refreshing the page will keep the last state of the app. In mobiles, refreshing the page
accidentally is very common. 
* Save the UI state of a page and navigate through the app preserving the query-params and then go back to the initial
page, and the last state will be automatically restored.
* If you want to bookmark or share a specific state of the application using the browser bookmark feature or using hiperlinks. 

Ng-ui2qp uses a declarative approach in conjunction with the fact that it's integrated with Angular Reactive Forms,
makes the lib intuitive and easy to use.
 
## Advantages
 
- Extends ReactiveForms API which makes it easy to understand and work with it.
- You can save whatever model(Object) to the QP of the URl. You could create complex nested forms and still use the lib to save it.
- Compatibility with Angular Validations.
- Possibility to add and remove dynamically properties of the defined model and will stay synchronized with the QPs.  
 
##Main Concepts
 
One of the strongest point that Ui2Qp has is that it **extends ReactiveForms classes**. Which makes it very easy to
understand and use if you are familiar with ReactiveForms API.
Here's an overview of how the Ui2Qp classes matches ReactiveForms:
 
| ReactiveForms | Ui2Qp | | 
| ----------- | ----------- | -----------|
| FormControl | Ui2QpFormControl | Extends | 
| FormGroup | Ui2QpFormGroup | Extends |
|     X     | Ui2QpRoot | Ui2Qp specific | 
| FormBuilder | Ui2QpBuilder | Doesn't extends |
 
Using Ui2Qp generally is only a matter of some few steps:

1. Provide a Router Adapter
2. Create a Ui2QpRoot object and set some settings.
3. Create a model structure that defines the parameters you want to synchronize with the URl.  
4. Bind the created model to the UI components responsible to update the value of each property of the model.
 
## Quick Start
 
1. Install the Lib.
 
```
ng add ng-ui2qp
```

2. Provide the default, or a custom Route Adapter ([Creating a Custom Router Adapter](#creating-a-custom-router-adapter)).
 
   You must provide a router adapter in the App's providers, here's an example:
   ```
   // AppModule providers array
   providers:[{ provide: QP_ROUTER_ADAPTER, useClass: Ui2QpDefaultRouterService }]
   ```

3. Inject the Ui2QpBuilder class in the component.

   ```
   constructor(private ui2QpBuilder: Ui2QpBuilder) {
   ...
   }
   ```
  
4. Instantiate a Root Object (Ui2QpRoot) and the model.
 
   Here's an example of creating a Ui2QpRoot instance that will automatically update the QP which each update of the
   model value and without creating a new entry in the browser history.
    
   **Notice that it's similar of creating a ReactiveForm**
 
   ```
   // Asumming the "ui2QpBuilder" is a reference to Ui2QpBuilder and it
   // was injected in the component where this code is used 
   const root: Ui2QpRoot  = this.ui2QpBuilder.root(
      { replaceState: true },
      {
       category: this.ui2QpBuilder.control(),
       price: this.ui2QpBuilder.control(),
      }
     );
   ```
    
  We could also have done it in these other ways too:
   
  ```
  const model = this.ui2QpBuilder.group(
      {
        category: this.ui2QpBuilder.control(),
        price: this.ui2QpBuilder.control(),
      });
  const root: Ui2QpRoot  = this.ui2QpBuilder.root(
      { replaceState: true },
      model
     );
  ```
   
  ```
  const model = this.ui2QpBuilder.group(
      {
        category: this.ui2QpBuilder.control(),
        price: this.ui2QpBuilder.control(),
      });
  const root: Ui2QpRoot  = this.ui2QpBuilder.root({autoUpdating: {enabled: true}, replaceState: false});
  // In this moment the model will be updated with the query-params values if they exist at that point in time.
  root.model = model;
  ```
   
5. The last step is binding the model with the template, like you'll do with ReactiveForms, here's an example.

   ```
   // Using the code from the example above this will be the corresponding code in the template
   // for binding the model with the template.
    <ng-container [formGroup]="root.model">
       <input matInput formControlName="category">
       <input matInput formControlName="price">     
    </ng-container>
   ```
   ****Note that is the same as using ReactiveForms****
   
   **Important:** The ReactiveFormsModule must be imported in the module.

## Configurations

We know that every use case could be different and therefore we've provided some configurations that allows you to
adjust it to your specific use cases. 

### Configuration Object

The configurations object should be passed to the Ui2QpRoot at the moment of its instantiation, but it's not required,
you only need to pass it if you want to change
any of the default configurations and only does configurations should be included.
 
Default configurations that will be used by Ui2QpRoot if they aren't changed:
  
```
{
  autoUpdating: {
     enabled: true,
     debounce: 500,
   },
  replaceState: true,
}
```
#### autoUpdating

The "autoUpdating" configuration defines if the Ui2QpRoot should automatically update the QP when the model's value
changes and the updating frequency of the QP after a model change. For that
matter that property is an object which contains two properties which are:

1. enabled: Defines if the autoUpdating feature it's enabled or not, by default it is enabled.
2. debounce: Defines how many milliseconds will be waited after the model has changed before taking the
model's value and updating the QP. If the ***autoUpdating*** setting is ***enabled***.

#### replaceState

Defines if the Ui2QpRoot will create a new state in the browser history every time the QPs are updated.

## Serializers and Deserializers

Serializers and Deserializers determine how the value of the Ui2QpFormControl is serialized into a string for the URL and, 
vice versa, how the parameter value is converted into the value used in the Ui2QpFormControl. You could creat your own
ones if needed and pass them to the Ui2QpFormControl.

### Built-in Serializers and Deserializers

We have created for you some Serializers and Deserializers for most of the common types. here are they:

| Type of Value | Type ID | 
| ----------- | ----------- |
| Number | number | 
| String | string |
| Array of number number | number-array |
| Array of string | string-array |

*Note*: Booleans are treated as numbers(false=0,true=1) if you want a different behavior feel free to create your own 
serializer and deserializer (check [Creating and registering your own custom Serializers and Deserializers]#creating-and-registering-your-own-custom-serializers-and-deserializers)). 

### Deserializers

The deserializer is no more than an object which has two properties, a function and a type. It's used for transforming 
the values retrieved from the QPs (which are always string) to the right value expected by the Ui2QpFormControl its bind to.

```
{
  type: string;
  deserializerFunc: (value: any, defaultValue: any) => any;
}
```
The "type" is for identifying the deserializer with a unique id that later will be used to find the right deserializer
for a control with the same type.
The "deserializerFunc" is a function that transforms the value retrieved from the QP to the right type, son can be set 
as a value to its corresponding Ui2QpFormControl within the model.
Here's the Number Deserializer's implementation, take it as an example:
```
{
  type: 'number',
  deserializerFunc: (value: any, defaultVal: any) => {
    const returnValue = parseInt(value);
    if (typeof returnValue !== 'number') {
      console.error(`The current value ${value} couldn't be deserialized, instead we used the default value`);
      return defaultVal;
    }
    return returnValue;
  },
}
```

As you can see in the above implementation the value it's been transformed from string to number using the "parseInt"
function, which later it's being return and used as a value for its corresponding Ui2QpFormControl. In addition, we are
handling here another case, what should happen if after applying the transformation still the values is not the correct,
or an error occur as a result of trying to apply the transformation? For that case we included in the params a default value,
so it can be used in case of something goes wrong or is not clear which value to return; going back to the previous ,
example if that "parseInt" returns a wrong value we show an error in the console, and we use the default value provided
instead and nothing stopped working :).

### Serializers

A serializer has a similar structure to a Deserializer, a type and a function, here it's:

```
{
  type: string;
  serializerFunc: (value: any) => string;
}
```

As you can see it's alike the Deserializer structure, the main difference is in the params that the serializeFunc takes 
and the return value. The reason is the serializers are executed at the moment of updating the QP and they act as a map
function that takes a value returned by a Ui2QpFormControl and maps it to string that is because the QPs only accept string alues.

This is the implementation of the default serializer provided by us:

```
{
  type: 'string',
  serializerFunc: (value: any) => {
    return value;
  },
}
```
As you can see in the above code the default serializer does not transform the value received, it just returns it.
That is wanted because all the values of an Angular form in normal use cases will be primitives types(number,string,
boolean or an array of them) which all are possible to set as QP without any kind of mapping needed.
In other cases of not primitive types you may need to define custom serializers who map the value retrieved from the 
form to string or other primitive types.
That will be the case for custom form controls which implement "ControlValueAccessor", and the value returned is not 
a primitive or an array of them.

***Note***: If you aren't using the Ui2QpDefaultRouterService, or the Angular router as a base router for changing the
QP you may need to define a serializer or serializers for cases in which a Ui2QpFormControl could return an array, a 
normal case will be a multiselect input which will return an array of selected values. 
If you are using the Angular Router like is the case of the Ui2QpDefaultRouterService then angular takes care of making
that mapping for you, so we strongly recommend using it :).

### Creating and registering your own custom Serializers and Deserializers

You may need to create custom Serializers and Deserializer for specific Ui2QpFormControl types or custom components that
implement "ControlValueAccessor" and don't use a primitive-based type for its value, in that case you may think on creating
new Serializers and Deserializers to handle the value mapping.

Here is an example of creating a custom Serializer and Deserializer for a DateTime picker([ng-pick-datetime](https://www.pmjs.com/package/ng-pick-datetime)),
component which implements "ControlValueAccessor", and its value's type is a Date object.

#### Steps

1. Define de type for the Serializer and Deserializer, it should be provided while creating a Ui2QpFormControl using the 
Ui2QpBuilder.

```
const dateTimePickerType = 'datetime-picker';
```

2. Create the custom Deserializer

```
const datetimePickerDeserializer: Deserializer = {
  type: dateTimePickerType,
  deserializerFunc: (value: string, defaultValue: Date) => {
    return new Date(value);    }
};
```

Notice how the value which is type string it's converted to a Date object since we've got it from the QPs.

3. Create the custom Serializer.

```
const datetimePickerSerializer: Serializer = {
  type: dateTimePickerType,
  serializerFunc: (value: Date) => {
     return value.toISOString();
  }
};
```

Notice how the value which is a Date it's been transformed to an ISO string format which is going to be used in the QP

4. Register both Serializer and Deserializer with the Ui2QpDeserializersService and Ui2QpSerializersService so can be
found by the Ui2QpBuilder while creating and initializing controls of the same type.

```
// Asumming Ui2QpDeserializersService and Ui2QpSerializersService were injected in the component's contructor
this.serializersService.register(datetimePickerSerializer);
this.deserializersService.register(datetimePickerDeserializer);
```

5. Usage example.

```
// Asumming all the previous steps were done and their code exist in the context of this example
this.model = this.ui2QpBuilder.group({
       // especifying the control's serializer and deserializer type, check step 1.
      datetime: this.ui2QpBuilder.control(dateTimePickerType),
    });
// Creating the root and changing some default settings
this.root = this.ui2QpBuilder.root({autoUpdating: {enabled: true}, replaceState: false});
// linking the model with the QP when
this.root.model = this.model;
```

That's it, its all you need to do for creating customs Serializer and Deserializer if you need it.

## Router

The Router is how the lib interacts with the browser's URL and the QPs, it's the middleware that takes care of setting
the QPs into the URL and retrieves them from it.
We've provided our own Router adapter which uses Angular Router in background to interact with the QPs, and we believe 
it's a good idea to use it instead of defining your own one, the reasons are: 

1. It's easy to use.
2. Has been proved extensibility 
3. Compatibility with the Framework.

We still believe that you may find some cases where the one provided by us it's not enough, so we've included a way
to create your own one and use it instead.

### Creating a Custom Router Adapter

1. Create a new service. 
2. Implement the interface "Ui2QpRouter" and implement the methods the interface has. Here is the code of the 
"Ui2QpDefaultRouterService" which is provided by us, but it's a good example:

```
@Injectable()
export class Ui2QpDefaultRouterService implements Ui2QpRouter {

  constructor(private router: Router, private activatedRoute: ActivatedRoute) {
  }

  public getUrl() {
    return this.router.url;
  }

  public getQueryParamMapObservable() {
    return this.activatedRoute.queryParamMap;
  }

  public getQueryParamMap(): Params {
    // The queryParamMap object returned by the activated router snapshot has an extra property
    // that encapsulates the actual queryParam object
    return (this.activatedRoute.snapshot.queryParamMap as any).params;
  }

  public navigate(queryParams: Params, replaceState: boolean): Promise<boolean> {
    return this.router.navigate([], {
      relativeTo: this.activatedRoute,
      queryParamsHandling: null,
      queryParams,
      replaceUrl: replaceState
    });
  }

  public getCurrentNavigation() {
    return this.router.getCurrentNavigation();
  }
}
```

3. Provide (normally in the app.module of your app) your new Service as the RouterAdapter to use, here is how:

 ```
 // AppModule providers array
 providers:[{ provide: QP_ROUTER_ADAPTER, useClass: <Your service name> }]
 ```

That's all you have to do for creating and using your own custom Router Adapter it's really easy, right?.

## Usage examples

We've created a set of examples in [stackblitz](https://stackblitz.com/edit/ng-ui2qp) which can help to understand 
and give a glance of what can be done with the ng-ui2qp.

## Warning

Please keep in mind that for compatibility matters between browsers you have to take care of the URL's length doesn't 
get beyond 2,083 characters because that could cause some unexpected behaviour in old browsers. From our experience 
this could happen if you are working with very deep nested forms or long texts.
Please read [this](https://www.quora.com/What-is-the-max-length-for-URLs-in-each-browser) form more information.
