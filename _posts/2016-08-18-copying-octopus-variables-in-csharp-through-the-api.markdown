---
author: Peter Gerritsen
comments: true
date: 2016-08-18 11:00:00+02:00
layout: post
slug: copying-octopus-variables-in-csharp-through-the-api
title: Copying Octopus variables in C# through the API
tags:
- Octopus
categories: Octopus
---

For a project I needed to move variables from a project in Octopus to a library variable set for reuse in other project. 
Because the source project contained over 100+ variables with different scoping, I didn't want to do this by hand.

Enter the Octopus API:

```csharp
namespace OctopusVariableCopier
{
    class Program
    {
        static void Main(string[] args)
        {
            var octopusUrl = "https://<enter server url>";
            var apiKey = "API-<enter API key";

            var octopusServer = new Octopus.Client.OctopusServerEndpoint(octopusUrl, apiKey);
            var repo = new Octopus.Client.OctopusRepository(octopusServer);

            var libraryVariableSetLib = repo.LibraryVariableSets.Get("LibraryVariableSets-61");
            var libraryVariableSet = repo.VariableSets.Get(libraryVariableSetLib.VariableSetId);
            var project = repo.Projects.Get("source-project-slug");
            
            var projectVariableSet = repo.VariableSets.Get(project.VariableSetId);
            foreach (var projectVariable in projectVariableSet.Variables) {
                if (projectVariable.Scope.ToString().Contains("Action =")) // skip step scoped variables 
                    continue;

                Console.WriteLine("{0} : {1}", projectVariable.Name, projectVariable.Value);

                libraryVariableSet.Variables.Add(projectVariable);                
            }

            repo.VariableSets.Modify(libraryVariableSet);   
            
            Console.ReadLine();
        }
    }
}
```

After this you can use the same structure to loop through the library variables to delete the variables from the project variable set. 

If later you need to check if the values from the source project are the same you can use the following check.

```csharp
var projectVariable = projectVariableSet.Variables.FirstOrDefault(x => x.Name == libraryVariable.Name && x.Scope.ToString() == libraryVariable.Scope.ToString());

if (projectVariable.Value != libraryVariable.Value)
	Console.WriteLine($"Variable: {projectVariable.Name} - Scope: {projectVariable.Scope} - Project value: {projectVariable.Value} - Library value: {libraryVariable.Value}"); 
```
