# jQuery Form Double-Submit Prevention
*Prevents consecutive form submissions of identical form values.*

Repetitive form submissions that would submit the identical form values are prevented, unless the form values are different to the previously submitted values.

This library is a simplified re-implementation of a user-agent behavior that should be natively supported by all major web browsers, but at this time, only Firefox has a built-in protection.


## Usage

To retain a clean separation of concerns, this library only supplies a [`submit` event handler](https://github.com/sun/jquery-form-submit-single/blob/master/src/jquery.form-submit-single.js).  It is up to you how you want to use it.

Simple usage example:

```html
<script type="text/javascript" src="src/jquery.form-submit-single.js"></script>
<script type="text/javascript">
jQuery(function ($) {
  $('body').on('submit.formSubmitSingle', 'form:not([method~="GET"])', $.onFormSubmitSingle);
});
</script>
```

1. The `submit` event bubbles up, you can bind it just once and delegate it to all forms on the page.
2. Namespace the registered event handler by using `submit.formSubmitSingle`.
3. Exclude forms that are using `method="GET"`.


## Background

Double-clicks on form buttons, or double-submits of forms, present a major problem for non-persistent/non-daemonized web applications, as they are technically not able to detect parallel requests:

The moment you click a form submit button, your browser issues a new (parallel) HTTP request to the web server.  The server-side script application does not have a _shared application state_; any information that may be contained in previous or parallel requests is unknown.

Consequently, two (or more) _parallel_ HTTP POST requests containing the identical data are processed independently of each other.  Depending on the action performed, this may result in an undesired or unexpected user experience.

Duplicate form submits can happen within milliseconds (a typical double-click, commonly performed by users on _Microsoft Windows_), but also in intervals of seconds (typically caused by a web site not "reacting" due to too much server load or expensive form processing operations).

In any case, it is the end-user on the client-side, who manually triggers multiple form submits of the identical POST data.

All major web browsers - except _Mozilla Firefox_ - do not implement a double-click/submit protection at this point.  Until that changes, web applications share the need for a custom protection to prevent multiple, identical form submissions.

A custom protection can only be implemented on the client-side via JavaScript.  We can't do anything about clients that have JavaScript disabled.

This library aims to provide a generic solution that can be (1) shared across projects, (2) improved by the wider web community, and ultimately (3) removed and marked as obsolete, as soon as major web browsers ship with a native solution.


## Anatomy

Form buttons may have further event handlers attached to them, not invoking an actual form submit (i.e., XHR, Ajax, etc).  A protection against multiple submits could break other scripts, which we want to avoid, of course.

As an attempt to resemble the built-in double-submit protection in Firefox, this implementation

1. Records the submitted form values via `jQuery.serialize()`.
2. Checks whether the form values have changed compared to a previously triggered form submit.
2. If the form values are identical, a consecutive form submit is prevented.  
   (If there are differences, a consecutive form submit is executed.)

A value-based approach ensures that the constraint is triggered for consecutive, identical form submissions only.

In case there are multiple submit buttons (e.g., _Preview_, _Save_, _Delete_), then the button's value itself is not part of the values being compared.  That is intentional, because once a HTTP request is in progress, the server-side processing state is unknown.

Compared to that, a button-based approach would (1) rely on [visible] buttons to exist where technically not required and (2) require more complex state management if there are multiple buttons in a form.

Lastly, all forms using `method="GET"` are idempotent by design and web standards:  Any amount of HTTP GET requests should yield the exact same response and result, so all forms being submitted via GET should be excluded from the protection. _(but it is up to you whether you want to follow this)_

To avoid an unnecessary dependency on `jQuery.data()`, previously submitted form values are stored in a `data-form-submit-single-last` attribute on the `<form>` element of the submitted form.


### Known limitations

1. Form buttons with event handlers that `preventDefault()` on their own do not receive a double-submit protection.  That is deemed to be fine, because such buttons typically trigger _reversible_ client-side or server-side operations that are local to the in-progress state of the form only.
2. Changed values in advanced form controls, such as file inputs, are not part of the form values being compared between form submits (due to `jQuery.serialize()`).  That is deemed to be acceptable, because:
    * If the user forgot to attach a file, then the size of HTTP payload will most likely be small enough to be fully passed to the server endpoint within (milli)seconds.
    * If the user mistakenly attached a wrong file and is technically versed enough to cancel the form submission (and HTTP payload) in order to attach a different file, then that edge-case is not supported here.
3. There does not seem to be a way to feature-detect the native double-submit protection of e.g. Firefox.


### On not manipulating the user interface

Be cautious with the idea of manipulating the user interface.  You will need to manage the state of all performed manipulations, which gets incredibly complex very fast.  Any other scripts inherently need to be aware of the facility and integrate with it, so they can react accordingly → a fully-fledged state tracking machine for forms, _overkill._

Also, _consistency:_  The built-in double-submit protection in Firefox has no visual indication for when that kicks in either — the browser is just simply smart enough to figure out that you did not intend to post the identical form values more than once under the hood.

Also, _reliability:_  The only way to provide reliable user interface feedback is to watch and track the actual state of the HTTP request.

This library is a work-around for a problem that is 100% in the domain of user agents and ought to be addressed there (following Firefox's lead), as opposed to client-side scripts.  Once other vendors are catching up, there's a good chance for a native facility and cross-browser user experience to arise, which will be consistent on all sites on the net, and not just yours.


## Credits

* Concept & original implementation: [@sun](https://twitter.com/tha_sun)
* JavaScript fu: [@nod_](https://twitter.com/nod_)


## License

Released under the MIT license; http://sun.mit-license.org/2013
