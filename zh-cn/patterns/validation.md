#Validation 

<br>

## Validation on non-string types

Matt:
> how to handle errors in binding
> It looks like there's no direct way to turn a conversion failure into a "validation error"

Ryan:
> re: binding/date validation, maybe you should bind to an intermediate string property on the vm and do validation over that? then you can do other typical validation things like surfacing an error message, and you're not limited to the type converter confines

Kent:
> if you can’t use a control specifically designed for datetimes (e.g. `DateTimePicker`) then I would bind to a string property (let’s call it `A`). Then have a get-only `DateTime` or `DateTime?` property derived from that (let’s call it `B`). When `A` changes, attempt to update `B`. If the parse fails, either leave `B` as `null` or set another property (`C`) to contain the validation error you want the view to display.


<br>


## Value Bounds Checking


Matt:
> Does anyone have a good way to deal with value bounds checking? for instance "username should be 3 to 12 characters" from the viewmodel.

Kent:
> my approach is normally to break into separate properties: `UserName` and `UserNameError` (which is `null` if there is no error). Then use the platform’s validation/error mechanism to bring to user’s attention

Brendan:
what Kent said
> excep we called it `UserNameValidator` and rolled a bit of interesting code to declaratively define the rules - and then had a `.Result` to inspect at the end

Dennis:
> I'm doing it exactly like @kentcb
