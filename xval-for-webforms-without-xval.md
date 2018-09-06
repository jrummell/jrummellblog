---
permalink: /xval-for-webforms-without-xval
title: xVal for WebForms without xVal
date: 2011-06-05 23:23:44
published: true
tags: xval-webforms
---


I’ve been working on [xVal for WebForms](http://xvalwebforms.codeplex.com/) without xVal in the [jQuery.Validate](http://xvalwebforms.codeplex.com/SourceControl/list/changesets?branch=jQuery.Validate) branch. So far I’ve got basic server and client side validation for most data annotations validation attributes and server side validation for [IValidatableObject](http://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.ivalidatableobject.aspx) implementers. The only challenging part so far was understanding how to serialize the validation rules for the jQuery Validate [add method](http://docs.jquery.com/Plugins/Validation/rules).

    $("#txtClientName").rules("add", { 
        required: true, 
        minlength: 5, 
        messages: { required: "Client name is required.", 
                    minlength: "Client name must be at least 5 characters." } 
    });

I finally decided to implement JavaScriptConverter, which is used with JavaScriptSerializer. The Serialize method is what takes the Rule collection and converts it into two separate dictionaries, one for rules and one for messages. This allows the JavaScriptSerializer to properly serialize the dictionaries for the add method options parameter.

{% raw %}
``` csharp
   public class RulesJavaScriptConverter : JavaScriptConverter
   {
      private readonly ReadOnlyCollection<Type> _supportedTypes =
          new ReadOnlyCollection<Type>(new List<Type>(new[] {typeof (RuleCollection)}));
   
      public override IEnumerable<Type> SupportedTypes
      {
          get { return _supportedTypes; }
      }
   
      public override object Deserialize(IDictionary<string, object> dictionary, Type type,
                                         JavaScriptSerializer serializer)
      {
          throw new NotSupportedException();
      }
   
      public override IDictionary<string, object> Serialize(object obj, JavaScriptSerializer serializer)
      {
          return Serialize(obj as RuleCollection, serializer);
      }
   
      public IDictionary<string, object> Serialize(RuleCollection rules, JavaScriptSerializer serializer)
      {
          if (rules == null)
          {
              throw new ArgumentNullException("rules");
          }
   
          Dictionary<string, object> options =
              rules.ToDictionary<Rule, string, object>(rule => rule.Name, rule => rule.Options);
          Dictionary<string, string> messages =
              rules.ToDictionary(rule => rule.Name, rule => rule.Message);
   
          Dictionary<string, object> result =
              new Dictionary<string, object>(options) {{"messages", messages}};
   
          return result;
      }
    }
```
{% endraw %}

And here’s how I’m using it.

``` csharp
    StringBuilder validationOptionsScript = new StringBuilder();
    validationOptionsScript.AppendFormat("$('#{0}').rules('add', ", _controlToValidateId);

    JavaScriptSerializer serializer = new JavaScriptSerializer();
    serializer.RegisterConverters(new[] {new RulesJavaScriptConverter()});
    serializer.Serialize(rules, validationOptionsScript);

    validationOptionsScript.AppendLine(");");
```

The next step will be figuring out validation groups. I also plan on renaming the project. My best idea so far is jQuery Validate.NET. It’s not the most creative, but it gets the point across.