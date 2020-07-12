# Shuup Onboarding

Add dynamic panels for onboarding purposes in Shuup (specially in Admin).

### Installation & Configuration

#### Install this package

Run `pip install shuup_onboarding` and  add `shuup_onboarding` to `INSTALLED_APPS`.

#### Implement your onboarding steps and add the onboarding middleware

Write some onboarding steps (any number you want) by implementing the `OnboardingStep` class, follow a simple example:

```py
class MyInfoStep(OnboardingStep):
    identifier = "my_step_id"
    title = "My Step Title"
    description = "Some description of the step"
    template_name = "my_app/step_template.jinja"

    def can_skip(self):
        return False

    def is_done(self):
        return self.context.storage.get("my_info")

    def is_visible(self):
        return True

    def get_form(self, **kwargs) -> forms.Form:
        return MyStepForm(**kwargs)

    def save(self, form):
        self.context.storage["my_info"] = form.cleaned_data["info"]

    def undo(self):
        self.context.storage.pop("my_info", None)
```

Then, you need to create an unique onboarding process identifier. For a sake of example, let's user `my_onboarding_process` as our onboarding identifier.

Now you need to override the base middleware class `BaseAdminOnboardingMiddleware` and configure your process id attribute:

```py
class MyCustomOnboardingMiddleware(BaseAdminOnboardingMiddleware):
    onboarding_process_id = "my_onboarding_process"
```

To make the onboarding process load your steps, you must use a special provides key:

```
onboarding_process:[YOUR_ONBOARDING_PROCESS-ID]
```

Finally, add your onboarding steps to the provides:


```py
class AppConfig(shuup.apps.AppConfig):
    provides = {
        "onboarding_process:my_onboarding_process": [
            "my_app.onboarding_steps.MyInfoStep",
            "my_app.onboarding_steps.FinalStep",
        ]
```

Done.

You can find a full working example at [Shuup Onboarding Example](https://github.com/chessbr/shuup-onboarding-sample).

## License

Open Software License version 3.0

## Copyright

Copyright (c) 2020 Christian Hess
