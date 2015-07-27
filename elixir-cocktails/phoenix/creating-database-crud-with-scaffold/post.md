# Creating CRUD app in Phoenix with phoenix.gen.html

## The problem

Suppose you want to create some basic CRUD app. You want to quickly and easily generate your views, templates and controllers instead of writing all the basic stuff by hand.

## The solution

The solution is to use one of Phoenix-specific `mix` tasks, which are
already available within a newly-generated Phoenix app.

### Generating model with simple CRUD

To generate a model and corresponding view, templates and controller we
simply invoke `mix phoenix.gen.html` task.

    mix phoenix.gen.html Post posts title body:text

`Post` above is the module name, whereas `posts` is gonna be used as name for resources and schema. Other arguments are field names. As type wasn't provided, fields are gonna have default type - string.

The files generated include:
- migration
- model
- controller
- templates
- view
- controller test
- model test

The generator did not add, however, appropriate routing entries. Let's
fix that. We are reminded about that (and the need to run migrations) every time we use
phoenix.gen.html. To add proper routes to `router.ex`,
let's edit our `web/router.ex` file:

    scope "/", CrudBasicApp do
      pipe_through :browser # Use the default browser stack

      get "/", PageController, :index
      resources "/posts", PostController #add this line
    end

### Migration

After adding resources to router, everything is almost ready. All that
is left is running migration on applications database.

    mix ecto.migrate

If you haven't created your database, now is a good moment for that.
Simply run:

    mix ecto.create

And run migration after that.

That's it! We are all set with our Post resource.
Feel free to start server and play a bit with managing posts.

    mix phoenix.server

Posts can be found at `http://localhost:4000/posts`.

### Adding related resource

Ok, our posts are great. But suppose now we want to comment on them. Each
comment is gonna include some username and content. We want to make
sure there is a relation between our models - a post may have many
corresponding comments, and comment belongs to just one post. So we
create a resource belonging to another one:

    mix phoenix.gen.html Comment comments name content post:belongs_to

The generator is doing great job here too. It took care of creating
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
info we passed while creating comment. We are gonna change this line in
`web/templates/comment/index.html.eex`:

    <td><%= comment.post %></td>

to this:

    <td><%= comment.post_id %></td>

If you create a few comments, you will see something is wrong. You choose the number, which is supposed to be
`post_id` in the dropdown, but it does nothing. In index, there is no post_id. Great... Looks like we still have to ensure new comment is connected to some post in controller.

Let's add following line to our `CommentController`:

    alias CrudBasicApp.Post

and change `create` action a little:

    new_post = Repo.get!(Post, comment_params["post_id"])
    new_comment = build(new_post, :comments)
    changeset = Comment.changeset(new_comment, comment_params)

Now creating comments works. You can change the rest of actions and templates the
same way.

## Extra tips

More mix tasks for generating files can be found in [Phoenix documentation](http://hexdocs.pm/phoenix/overview.html#modules_summary).

Generated templates assign Bootstrap classes to page elements. You can
find more about [Bootstrap here](http://getbootstrap.com/).


Contributors:

[Dominika Kruk](mailto:dominika.kruk@amberbit.com)
