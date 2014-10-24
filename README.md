# Fix Security for Ivan.

## Ivan's Terrible Blog is at it again. This time with security holes left gaping wide for XXS attacks and SQL injection attacks.

### Here's how I plugged them up:

* SQL injection fix:
I looked for spots in the code where the search box was vulnerable. Taking hints from Railscast #178, I found a problem with the post model (post.rb). Take a look at line 12:
```
includes(comments: :replies).where("title like '%#{search}%'")
```
The post parameters on post titles are put directly in the SQL string, which is BAAAAAD, and vulnerable to SQL injection.

I changed line 12 to make it secure:
```
includes(comments: :replies).where("title like ?", "%#{search}%")
```
The addition of the question mark syntax, gets rid of putting the post parameters directly into the string. after the 'title like', on that parameter. The where method now takes an array, and assigns attributes to the question mark. This will escape special characters that might be used in an attack.

* Cross Site Scripting (XXS) fix:
This one was a little more difficult to understand. So I followed along this morning's discussion on Reed's solution. Again the consideration is on the search form, but this time we're looking at _search_form.html.erb in the views/posts folder.
Line 19 change:
```
<%= content_tag :div, "data-#{params[:status] == 'published' ? 'published' : 'unpublished'}" => @posts.size do %>
```
Putting the published ? statement in whitelists what is allowed.

Line 20 change:
```
Number of <%= sanitize params[:status] %> results shown: <%= @posts.size if @posts %>
```
The change that was made here was the addition of the sanitize method on params[:status].

Other changes made-
Line 7 of application_controller.rb
```
response.headers['X-XSS-Protection'] = '0'
```
...was changed to:
```
response.headers['X-XSS-Protection'] = '1'
```

Additionally, in the Gemfile, where the Rails version is called out, the change was made on the version from 3.2.13 to just 3.2 - so that the minor version updates would be included automatically, when newer versions than 3.2.13 become (became) available.

I also added the brakeman gem and ran a report to make sure no 'high' importance issues were showing up anymore.


