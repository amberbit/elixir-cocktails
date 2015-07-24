# Creating CRUD app in Phoenix with phoenix.gen.html

## The problem

Suppose you want to create some basic CRUD app. You want to quickly and easily generate your views, template and controllers instead of writing all the basic stuff by hand.

## The solution

The solution is to use one of Phoenix-specific mix tasks, which are
already available within a newly-generated Phoenix app.

### Generating model with simple CRUD

We are going to use Ecto with Postgrex - adapter for PostgreSQL.
To generate a model and corresponding view, templates and controller
simply invoke mix phoenix.gen.html task.

    mix phoenix.gen.html Post posts title body

`Post` above is the module name, whereas `posts` is gonna be used as name for resources and schema, which will store our posts. Other arguments are field names.

The files generated include:
- migration
- model
- controller
- templates
- view
- controller test
- model test

Now it's time for adding proper routes in our application router
and migrating database. We are reminded about that every time we use
phoenix.gen.html, so it's not a problem. To add proper routes to router,
let's simply paste the generated line in browser scope in `web/router.ex`.

    scope "/", CrudBasicApp do
      pipe_through :browser # Use the default browser stack

      get "/", PageController, :index
      resources "/posts", PostController #add this line
   end

### Migration

After adding proper line in router, everything is almost ready. All that
is left is running migration on applications database.

    mix ecto.migrate

If you haven't created your database, now is a good moment for that.
Simply run:

    mix ecto.create

And migrate after that.

That's it! We are all set with our Post resource.
Feel free to start server and play a bit with managing posts.

    mix phoenix.server

Posts can be found at `localhost:4000\posts`.

### Adding related resource

Ok, our posts are great. But suppose now we want to comment on them. Each
comment is gonna include some username and content.
The mix task we previously used was pretty neat but now we want to make
sure there is a relation between our models - a post may have many
corresponding comments, and comment belongs to just one post. We can
create such model using the generator:

    mix phoenix.gen.html Comment comments name content post:belongs_to

As always, generator is doing great job. It took care of creating
relation with posts in database, put index on post_id and created proper
views, templates, etc. The only thing we have to do manually at this
point is inform our posts, that now they have comments! Let's edit the
post model now.

  schema "posts" do
    has_many :comments, CrudBasicApp.Comment
    field :title, :string
    field :body, :string

    timestamps
  end

That's it! Now it's possible to create a comment to post.

### Linking new comment to Post

Let's change a little our generated template, so we will see exactly the
info we passed creating comment. We are gonna change this line in
`web/templates/comment/index.html.eex`

  <td><%= comment.post %></td>

to this:

  <td><%= comment.post_id %></td>

If you create now a few comments, you will see something is wrong. You choose the number, which is supposed to be
post_id in the dropdown, but it does nothing. Great... We still have to ensure new comment is connected to some post in controller.

Let's add following line to our `CommentController`:
  alias CrudBasicApp.Post

and change `create` action a little:

  new_post = Repo.get!(Post, comment_params["post_id"])
  new_comment = build(new_post, :comments)
  changeset = Comment.changeset(new_comment, comment_params)

Now creating comments works. You can change the rest of actions and templates the
same way. 

### Complete example

The complete example application for this recipe is [available on
GitHub]().
The relevant commit changes can [be found
here]().

## Extra tips

More mix tasks for generating files can be found on http://hexdocs.pm/phoenix/overview.html#modules_summary

Generated templates assign Bootstrap classes to page elements. You can
find more about Bootstrap here: http://getbootstrap.com/


Contributors:

[Dominika Kruk](mailto:dominika.kruk@amberbit.com), [Hubert Łępicki](mailto:hubert.lepicki@amberbit.com)
