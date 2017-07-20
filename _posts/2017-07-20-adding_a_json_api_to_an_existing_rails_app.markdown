---
layout: post
title:  "Adding a JSON API to an existing Rails app "
date:   2017-07-20 18:39:57 +0000
---


I have been working on updating my previous Rails app with some javascript/json functionality. [Check out](https://andrewjford.github.io/2017/07/04/auditrequest/) my last post to learn more about this app. My first target was the comments section.

![Comments section](https://i.imgbox.com/xUeFUIOl.png)

The comments section was an area that could benefit from Ajax postings and page updates. Having the new comment display immediately on the page without reload would work well, notably when the comment section was long. This would be done through an Ajax call to the JSON API. One of my focuses when implementing this was to maintain the functionality of the page even without javascript.

After setting up the comment serializer for the json endpoint, I had to edit the comments controller to handle the new JSON API endpoint as well as the existing html one. Below is this code in the create action for comments.

```
respond_to do |format|
  format.html {redirect_to project_request_path(params[:project_id],params[:request_id])}
	format.json {render json: @comment, status: 201}
end
```

In the above, Rails executes code based on the data type requested. If html is requested (the default when the comment form is submitted), a path redirect is executed. In this case it redirects to the project_request path which is just the page that shows comments (screencapped above). If json is requested, then json is rendered for the comment that was created.

So the controller is then setup to respond to both html and json on comment creation. The javascript makes requests specifically to the json endpoint rather than the html. Below is the function that is called when javascript hijacks the add comment form submittal.

```
function addComment(form){
  var formValues = form.serialize();
  $.post(form.attr('action'), formValues, showComment, "json");
}
```

Notice the last argument in the jQuery post call is "json". This is stating that the dataType should be json, communicating to the controller to respond with json rather than html. The first argument is the http path, the second is the serialized form, and the third is the callback function. This callback 'showComment' then goes on to take the returned json and add a new comment to the DOM without reloading the page.

In the case of deleting comments using Ajax, I used the more explicit jQuery method below.

```
$.ajax({
  url: element.attributes.href.value,
  type: 'delete',
  beforeSend: function(xhr) {xhr.setRequestHeader('X-CSRF-Token', $('meta[name="csrf-     token"]').attr('content'))},
  dataType: "json",
  success: function(resp){
    ...
	}
})
```

This method is the basis of the $.post() method used before. It allows one to pass more settings than what the shorthand $.post allows. Here I pass a 'delete' type and then the important 'json' dataType. Specifying the dataType lets the Destroy action in the controller know I want to the JSON endpoint rather than the html.
