# Creating form in Phoenix without database table

## The problem

Suppose you wish to have some form in your app. It could be a
contact form, or maybe a search form. You need to provide a couple
validations on fields, but don't want the form to have any connection
with database tables.

## The solution

The solution is to use `Ecto.Changeset` with `Ecto.Schema`, which are
modules included normally within `Ecto.Model`. Using them, you will be able to create a model and a changeset without
any database table.

### Creating model with no database connection

We are going to create a simple contact form, with "First name", "Last
name", "Email" and "Content" fields.
Let's start by defining our model for a simple contact form.

    defmodule FormTestApp.Form do
      use Ecto.Schema

      @primary_key false
      schema "non_db_form.contact" do
        field :first_name, :string
        field :last_name, :string
        field :email, :string
        field :content, :string
      end
    end

You can pass anything you want in place of `non_db_form.contact`. For
models which have corresponding tables in database, table name would be expected
here.

By setting `@primary_key` to false, we ensure that no primary key will
be created by schema, as we don't need one.
Notice that, instead of using whole `Ecto.Model`, we added only
`Ecto.Schema`, as we don't really need functionality provided by other modules included in `Ecto.Model`. Now we can start
working on a changeset.

### Adding a changeset

The `Ecto.Changeset` module is responsible for casting params
and performing validations.
Let's add this line to our module:

    import Ecto.Changeset

And create our changeset the same way we would for a standard, connected to database model.

    @required_fields ~w(last_name email content)
    @optional_fields ~w(first_name)

    def changeset(model, params \\ :empty) do
      cast(model, params, @required_fields, @optional_fields)
        |> validate_length(:last_name, min: 3)
        |> validate_length(:first_name, max: 3)
        |> validate_format(:email, ~r/@[a-zA-Z0-9]+\.[a-zA-Z]+/)
    end

The whole reason why we needed to create a model schema was to provide first
argument for changeset function. Our Form module is now ready.

### Creating a form with `form_for` helper

Finally, we can create our contact form. We will show the form at index page, which was generated automatically with `phoenix.start`.
Paste following into `web/templates/page/index.html`.

    <div class="jumbotron">

      <%= form_for @contact_form, page_path(@conn, :create), fn f -> %>

        <%= if f.errors != [] do %>
          <div class="alert alert-danger">
            <p>Oops, something went wrong! Please check the errors below:</p>
            <ul>
              <%= for {attr, message} <- f.errors do %>
                <li><%= humanize(attr) %> <%= message %></li>
              <% end %>
            </ul>
          </div>
        <% end %>

        <div class= "form-group">
          <%= label f, :first_name, "First name"%>
          <%= text_input f, :first_name %>
        </div>
        <div class= "form-group">
          <%= label f, :last_name, "Last name" %>
          <%= text_input f, :last_name %>
        </div>
        <div class= "form-group">
          <%= label f, :email, "Email" %>
          <%= email_input f, :email %>
        </div>
        <div class= "form-group">
          <%= label f, :content, "Content" %>
          <%= textarea f, :content %>
        </div>
        <%= submit "Send" %>

      <% end %>

    </div>

We can't see our form yet, because `@contact_form` is undefined. Let's fix that. Paste following in place of default index action of your controller (`web/controllers/page_controller.ex`):

    def index(conn, _params) do
      contact_form = Form.changeset(%Form{})
      render(conn, "index.html", contact_form: contact_form)
    end

In order to refer to our FormTestApp.Form as "Form" we need to alias it.
Just paste following line in controller:

    alias FormTestApp.Form

Now you can start server and see your simple form. You won't however be able
to submit
it yet, as we haven't defined proper route and action for that.

### Providing action for submit

As you can see, the form will be sent as a POST request to our
applications `page_path`. First we need to provide a proper
route in our `web/router.ex`. Let's add it to browser
scope:

    post "/", PageController, :create

The only thing left to do is adding action to controller.

    def create(conn, %{"form" => form_params}) do
      changeset = Form.changeset(%Form{}, form_params)

      if changeset.valid? do
        conn
        |> put_flash(:info, "You have passed all the validations.")
        |> render "index.html", contact_form: Form.changeset(%Form{})
      else
        render conn, "index.html", contact_form: changeset
      end
    end

Remember that you need to use `plug :scrub_params` in your controller,
or else all params will be always validated as present (making
validation on required params useless). Just paste following line in your
controller.

    plug :scrub_params, "form" when action in [:create]

If you are creating a real form for your app, you probably want to do
something with users input once the changeset proves valid. Maybe you
want to send it as an email, or further process the data provided. As this is
just a simple example, the only thing we added is a flash message
informing about changeset being valid.
And that's it! Our simple form with no database table is ready to use.

## Extra tips

You can find more about modules we used in Phoenix documentation on [Ecto.Schema](http://hexdocs.pm/ecto/Ecto.Schema.html) and [Ecto.Changeset](http://hexdocs.pm/phoenix/overview.html#modules_summary).

If you need to perform some unusual or complicated validations, remember
that you can write custom validators and pass them to changeset using
[validate_change](http://hexdocs.pm/ecto/Ecto.Changeset.html#validate_change/3) functions.


Contributors:

[Dominika Kruk](mailto:dominika.kruk@amberbit.com)
