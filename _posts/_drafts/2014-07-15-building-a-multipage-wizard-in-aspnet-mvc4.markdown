---
author: Peter Gerritsen
comments: true
date: 2014-07-15 19:43:11+00:00
layout: post
slug: building-a-resumable-multipage-wizard-in-aspnet-mvc4-part-1
title: Building a resumable multi-page wizard in ASP.Net MVC4 part 1
tags:
- ASP.Net
- jQuery
categories: [.Net, JavaScript]
---

For a b2b portal for a customer we needed to build a form that would be presented in the form of a multi-page wizard.
The user needed to be able to navigate through and resume the wizard at a later time without being required to meet all validation rules as long as they don't submit the form. This post (and following posts) will outline the steps taken to create such a wizard. The examples are simplified versions and not the real code. 

The example will be a form where the user can provide some basic profile info and interests and some sections in a later step will only be shown when a user has certain interests.  

The mechanism is based on a post by Bipin Joshi: [http://www.dotnetbips.com/articles/9a5fe277-6e7e-43e5-8408-a28ff5be7801.aspx](http://www.dotnetbips.com/articles/9a5fe277-6e7e-43e5-8408-a28ff5be7801.aspx)

##Data storage

The data for each step will be stored in a class for each step:

```csharp
public class NameAndEmailStep : WizardStepBase<StepCode> {
	
	[Required]
	public string FirstName {get;set;}
	
	[Required]
	public string LastName {get;set;}
	
	[Required]
	[Email]
	public string EmailAddress {get;set;}

}

``` 

As you can see, we will use DataAnnotations to set validation rules for the properties.

The class derives from a generic base class and uses an enum to specify which step it represents:

```csharp
public abstract class WizardStepBase<T> : ModelBase {
    public abstract bool IsValid(out List<ValidationResult> validationResults);

    public abstract string StepTitle { get; }
    
    public abstract T Step { get; }
}

public enum StepCode {
    NameAndEmail,
    PersonalInterests,
	Sports,
	Hobbies
}
```

All data step classes will then be used in a container class. This will be serialized and stored in the database when the user navigates through the wizard:

```csharp
public class WizardContainer {
	//Steps data
	
	public NameAndEmailStep NameAndEmail {get;set;}

	public PersonalInterestsStep PersonalInterests {get;set;}

	public SportsStep Sports {get;set;}

	public HobbiesStep Hobbies {get;set;}
}

``` 

##View

For each step we'll create a view. The model class of this view will be the class for the step:

```html
@model WizardExample.NameAndEmailStep

<div>

    <h2>Name and Email</h2>

    @using (Html.BeginForm("Index", "WizardExample", FormMethod.Post, new { id = "wizardstepform" })) {
        @Html.AntiForgeryToken()
        <input type="hidden" name="step" value="@Model.Step.ToString()" />
        
        <div class="errorMessages">
            <ul data-bind="foreach: ErrorMessages">
                <li data-bind="text: $data"></li>
            </ul>
        </div>

        <div class="content-lines">
            <div class="editor-line">
                <div class="editor-label">
                    @Html.LabelFor(model => model.Firstname): *
                </div>
                <div class="editor-field input300">
                    @Html.EditorFor(model => model.FirstName)
                    @Html.ValidationMessageFor(model => model.FirstName)
                </div>
            </div>
            <div class="editor-line">
                <div class="editor-label">
                    @Html.LabelFor(model => model.LastName): *
                </div>
                <div class="editor-field input300">
                    @Html.EditorFor(model => model.Lastname)
                    @Html.ValidationMessageFor(model => model.LastName)
                </div>
            </div>
            <div class="editor-line">
                <div class="editor-label">
                    @Html.LabelFor(model => model.EmailAddress): *
                </div>
                <div class="editor-field input100">
                    @Html.EditorFor(model => model.EmailAddress)
                    @Html.ValidationMessageFor(model => model.EmailAddress)
                </div>
            </div>
		</div>

		<p>
            <input type="submit" name="prevBtn" value="Previous" class="cancel" />
            <input type="submit" id="nextBtn" name="nextBtn" value="Next" class="cancel" />
        </p>
	}
</div>
```

As you can see the submit buttons have a class 'cancel', this prevents the unobtrusive javascript validation from going off and blocking the submit action. On the first step the 'Previous' button wouldn't be there offcourse, but this is an example.

##Controller

The controller contains two actions, both called Index with a HttpGet or HttpPost attribute, to handle the requests: 

```csharp
public class InschrijvingWizardController : Controller {
	[HttpGet]
	public ActionResult Index(string step, string id) {
	    if (!string.IsNullOrEmpty(id))
	        Session["WizardDataId"] = id;
	
	    var data = GetWizardData();
	    var stepVal = StepCode.NameAndEmail;
	    if (!string.IsNullOrEmpty(step))
	        stepVal = (StepCode)Enum.Parse(typeof(StepCode), step);	
	    
	    return View(stepVal.ToString(), data.GetCurrentStep(stepVal));
	}
	
	[HttpPost]
	public ActionResult Index(string step, string nextstep, string prevBtn, string nextBtn, string submitWizardBtn) {
	    var data = GetWizardData();
	    var stepVal = (StepCode)Enum.Parse(typeof(StepCode), step);
	
	    if (stepVal == StepCode.NameAndEmail) {
	        TryUpdateModel(data.NameAndEmail);
	        data.NameAndEmail.RunValidation = true;
	        nextstep = StepCode.PersonalInterests.ToString();	        
	    }
	    if (stepVal == StepCode.PersonalInterests) {
	        TryUpdateModel(data.PersonalInterests);
	        data.Reden.RunValidation = true;
	        if (string.IsNullOrEmpty(nextstep)) {
                if (!string.IsNullOrEmpty(nextBtn) && string.IsNullOrEmpty(prevBtn))
                    nextstep = StepCode.NameAndEmail.ToString();
                else
                    nextstep = StepCode.Sports.ToString();	            
	        }
	    }
		if (stepVal == StepCode.Sports) {
	        TryUpdateModel(data.Sports);
	        data.Reden.RunValidation = true;
	        if (string.IsNullOrEmpty(nextstep)) {
                if (!string.IsNullOrEmpty(nextBtn) && string.IsNullOrEmpty(prevBtn))
                    nextstep = StepCode.PersonalInterests.ToString();
                else
                    nextstep = StepCode.Hobbies.ToString();	            
	        }
	    }
		if (stepVal == StepCode.Hobbies) {
	        TryUpdateModel(data.Hobbies);
	        data.Reden.RunValidation = true;
	        if (string.IsNullOrEmpty(nextstep)) {
                if (!string.IsNullOrEmpty(prevBtn))
                    nextstep = StepCode.Sports.ToString();
                else if (!string.IsNullOrEmpty(submitWizardBtn)) {
                     // Store data
                     SaveWizardData(data);
					 // Process on server
				 	PortaalServiceCommunicator.ProcessWizard(Session["WizardDataId"] as string);
					return RedirectToAction("WizardComplete");
				}           
	        }
	    }
		

		SaveWizardData(data);

        return View(nextstep, data.GetCurrentStep((StepCode)Enum.Parse(typeof(StepCode), nextstep)));
	}

	public ActionResult WizardComplete(){
		return View("WizardComplete");	// Basic view
	}


	private WizardContainer GetWizardData() {
		if (Session["WizardDataId"] == null) {
	        return new WizardContainer() {
				NameAndEmail = new NameAndEmail(),
				PersonalInterests = new PersonalInterests(),
				Sports = new Sports(),
				Hobbies = new Hobbies()
			}
		} else {
	        var id = (string)Session["WizardDataId"];
	        return PortaalServiceCommunicator.GetPortaalData<WizardContainer>(id);
	    }
	}
	
	private void SaveWizardData(WizardContainer data) {
	    var id = "";
	    if (Session["WizardDataId"] != null) {
	        id = (string)Session["WizardDataId"];
	    } else {
	        id = Guid.NewGuid().ToString();
	        Session["WizardDataId"] = id;
	    }
   		PortaalServiceCommunicator.StorePortaalData<WizardContainer>(data, id, DataValueType.Json);
	}

```

The controller uses the TryUpdateModel (method of the Controller base class)  method to update just the data for the current step in the container class. All other data will be left untouched. We have two methods to retrieve and store data (which is done through the PortaalCommunicator. This performs the (de)serialization using the JSON.Net package and retrieves/stores the data through a WCF service in our case. You can easily substitute this with direct database storage or, for instance, a Redis  service. 

The navigation is determined through the values of *nextStep*, *prevBtn* and *nextBtn*. When *nextStep* has a value, the wizard skips to a specific step, otherwise it checks if the next of previous step is requested.

We now have a basic multi-page wizard, that will store data in between requests. When submitting on the final step page the data is stored and the service is called to process that data.

The wizard doesn't validate the data yet, so we'll add that in the next section.

##Validation 

Each step class implements the base class WizardData. Validation for each step is performed through the abstract method 'IsValid'. This will return a boolean to indicate if the data is valid and provide a list of ValidationResults.   