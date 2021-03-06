# Custom responses

### Overview

Sails apps come bundled with several pre-configured _responses_ that can be called from [action code](https://sailsjs.com/documentation/concepts/actions-and-controllers).  These default responses can handle situations like &ldquo;resource not found&rdquo; (the [`notFound` response](https://sailsjs.com/documentation/reference/response-res/res-not-found)) and &ldquo;internal server error&rdquo; (the [`serverError` response](https://sailsjs.com/documentation/reference/response-res/res-server-error)).  If your app needs to modify the way that the default responses work, or create new responses altogether, you can do so by adding files to the `api/responses` folder.

> Note: `api/responses` is not generated by default in new Sails apps, so you&rsquo;ll have to add it yourself if you want to add / customize responses.

### Using responses

As a quick example, consider the following action:

```javascript
getProfile: function(req, res) {

  // Look up the currently logged-in user's record from the database.
  User.findOne({ id: req.session.userId }).exec(function(err, user) {
    if (err) {
      res.status(500);
      return res.view('500', {data: err});
    }
    
    return res.json(user);
  });
}
```

This code handles a database error by sending a 500 error status and sending the error data to a view to be displayed.  However, this code has several drawbacks, primarily:

*  The response isn't *content-negotiated*; if the client is expecting a JSON response, they're out of luck
*  The response *reveals too much* about the error -- in production, it'd be best to just log the error to the terminal
*  It isn't *normalized*; even if we dealt with the other bullet points above, the code is specific to this action, and we'd have to work hard to keep the exact same format for error handling everywhere
*  It isn't *abstracted*; if we wanted to use a similar approach elsewhere, we'd have to copy / paste the code


Now, consider this replacement:

```javascript
getProfile: function(req, res) {

  // Look up the currently logged-in user's record from the database.
  User.findOne({ id: req.session.userId }).exec(function(err, user) {
    if (err) { return res.serverError(err); }
    return res.json(user);
  });
}
```


This approach has many advantages:

 - More concise
 - Error payloads are normalized
 - Production vs. development logging is taken into account
 - Error codes are consistent
 - Content negotiation (JSON vs HTML) is taken care of
 - API tweaks can be done in one quick edit to the appropriate generic response file


### Response methods and files

Any `.js` file saved in the `api/responses/` folder can be executed by calling `res.thatFileName()`.  For example, `api/responses/insufficientFunds.js` can be executed with a call to `res.insufficientFunds()`.

##### Accessing `req`, `res`, and `sails`

The request and response objects are available inside of a custom response as `this.req` and `this.res`.  This allows the actual response function to take arbitrary parameters.  For example: 

```javascript
return res.insufficientFunds(err, { happenedDuring: 'signup' });
```

And the implementation of the custom response might look something like this:

```javascript
module.exports = function insufficientFunds(err, extraInfo){
  
  var req = this.req;
  var res = this.res;
  var sails = req._sails;
  
  var newError = new Error('Insufficient funds');
  newError.raw = err;
  _.extend(newError, extraInfo);
  
  sails.log.verbose('Sent "Insufficient funds" response.');

  return res.badRequest(newError);

}
```



### Built-in responses

All Sails apps have several pre-configured responses like [`res.serverError()`](https://sailsjs.com/documentation/reference/response-res/res-server-error) and [`res.notFound()`](https://sailsjs.com/documentation/reference/response-res/res-not-found) that can be used even if they don&rsquo;t have corresponding files in `api/responses/`.

Any of the default responses may be overridden by adding a file with the same name to `api/responses/` in your app (e.g. `api/responses/serverError.js`).

> You can use the [Sails command-line tool](https://sailsjs.com/documentation/reference/command-line-interface/sails-generate) as a shortcut for doing this.
>
> For example:
>
>```bash
>sails generate response serverError
>```
>



<docmeta name="displayName" value="Custom responses">
