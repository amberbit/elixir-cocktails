# Reading and writing cookies in Phoenix with Plug

## The problem

Suppose you want to store some temporarily key:value data, based on user
selection or preferences. You want that data to be accessible after page refresh, until some future expiration date. 

## The solution

Using browser cookies is the usual way of solving those sort of problems. We are going to use ([`Plug.Conn`](http://hexdocs.pm/plug/Plug.Conn.html)) to read and write cookies from our phoenix application.

### Writing cookies

To write a cookie, we need to use `Plug.Conn` function [`put_resp_cookie(conn, key, value, opts \\ [])`](http://hexdocs.pm/plug/Plug.Conn.html#put_resp_cookie/4). As an example, let's use basic `index` action

##### Temporary cookies

By default, no expiration time is set, which means the cookies will
be cleared when user closes the browser. Let's set two temporary/session cookies:

    def index(conn, _params) do
        conn = Plug.Conn.put_resp_cookie(conn, "first_cookie_key", "first_cookie_value")
        conn = Plug.Conn.put_resp_cookie(conn, "second_cookie_key", "second_cookie_value")
        render conn, "index.html"
    end

This solution works properly, but we could make it look better. More "elixir way" would be to use pipeline operator:

    def index(conn, _params) do
        conn
            |> put_resp_cookie("first_cookie_key", "first_cookie_value")
            |> put_resp_cookie("second_cookie_value", "second_cookie_value")
            |> render "index.html"
    end

And this is it! We have already saved our cookies in the browser.

### Reading cookies

To read a cookie, you simply access `conn.cookies`, the data is already
fetched, parsed and provided to you by Phoenix. For example, let's show the value of a cookie on a page:

###### Define proper function in your `web/views/page_view.ex` file

    defmodule App.PageView do
        use App.Web, :view

        def cookies(conn, cookie_name) do
            conn.cookies[cookie_name]
        end
    end

###### Call it in `web/templates/page/index.html.eex` template

        <%= cookies(conn, "first_cookie_key") %>
        <%= cookies(conn, "second_cookie_key") %>

##### Complete example

The complete example application for this recipe is [available on
GitHub](https://github.com/amberbit/elixir_cocktail_reading_and_writing_cookies).
The relevant commit changes can [be found
here](https://github.com/amberbit/elixir_cocktail_reading_and_writing_cookies/commit/ad88d823082cd29dd1975f15e3e1b3ac69b4388c).

##### Cookie expiration time and persistent cookies

To define expiration time, we need to add optional parameter `max_age`:

    def index(conn, _params) do
        time_in_secs_from_now = 7*24*60*60 # a week from now
        conn
            |> put_resp_cookie("first_cookie_key", "first_cookie_value", max_age: time_in_secs_from_now)
            |> put_resp_cookie("second_cookie_value", "second_cookie_value", max_age: 24*60*60)
            |> render "index.html"
    end

If we will set `max_age` with value `0`. You cannot specify infinite
expiration time for cookies, but if you want to make the cookie really
persistent, you can set it's expiration date to a few years from now.

## Extra tips

More `put_resp_cookies` optional parameters you can find on http://hexdocs.pm/plug/Plug.Conn.html#put_resp_cookie/4

You can use `Timex` Elixir dependency library, for easier setup of expiration time. Add `{:timex, "~> 0.16.2"}` to project `mix.exs` file which allows you to use `Date`, `Time`, `DateTime` and many other modules that will help you handle time conversions. In our case, we can use an example:

    use Timex

    expire_date = Date.now |> Date.add(Time.to_timestamp(7, :days))
    secs_to_expire_date = Date.now |> Date.diff(date_to, :secs)

    conn |> put_resp_cookie("first_cookie_key", "first_cookie_value", max_age: secs_to_expire_date)

Contributors:

[Rafał Maksymczuk](mailto:rafal.maksymczuk@amberbit.com)

[Hubert Łępicki](mailto:hubert.lepicki@amberbit.com)

