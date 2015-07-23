# Reading and writing cookies in Phoenix with Plug

## The problem

Suppose you want to store some temporarily key:value data, based on user
selection or preferences. You want, that data, to be accessible after page refresh, until they reach their expiration date. Using the browser cookies, is the simplest way, you are possible to do that kind of things.

## The solution

We are going to use connection Plug([`Plug.Conn`](http://hexdocs.pm/plug/Plug.Conn.html)).

### Writing cookies

To write cookies, we need to use `Plug.Conn` function `put_resp_cookie(conn, key, value, opts \\ [])`. As an example, let's use basic `index` action

##### Temporarily/session cookies
By default, expiration time is set to browser session life time

    def index(conn, _params) do
        conn = Plug.Conn.put_resp_cookie(conn, "first_cookie_key", "first_cookie_value")
        conn = Plug.Conn.put_resp_cookie(conn, "second_cookie_key", "second_cookie_value")
        render conn, "index.html"
    end

This solution will work properly, but we could make it look better, more "elixir way". Let's use pipeline operator:

    def index(conn, _params) do
        conn
            |> put_resp_cookie("first_cookie_key", "first_cookie_value")
            |> put_resp_cookie("second_cookie_value", "second_cookie_value")
            |> render "index.html"
    end

And this is it! We have already saved our cookie in the browser.

##### Expiration time cookies
To define expiration time, we need to add optional parameter `max_age`:

    def index(conn, _params) do
        time_in_secs_from_now = 7*24*60*60 # a week from now
        conn
            |> put_resp_cookie("first_cookie_key", "first_cookie_value", max_age: time_in_secs_from_now)
            |> put_resp_cookie("second_cookie_value", "second_cookie_value", max_age: 24*60*60)
            |> render "index.html"
    end

If we will set `max_age` with value `0`. The cookie will be deleted.

### Reading cookies

To read cookie, you need to use your Plug connection instance. For example, let's show it in the view:

###### Define proper function in your `web/views/page_view.ex` file

    defmodule App.PageView do
        use App.Web, :view

        def cookies(conn, cookie_name) do
            conn.cookies[cookie_name]
        end
    end

###### Call it in `web/templates/page/index.html.eex` template
        <%= cookies(conn, "cookie_name") %>

## Extra tips
More `put_resp_cookies` optional parameters you can find on http://hexdocs.pm/plug/Plug.Conn.html#put_resp_cookie/4

You can use `Timex` Elixir dependency library, for easier setup of expiration time. Add `{:timex, "~> 0.16.2"}` to project `mix.exs` file which allows you to use `Date`, `Time`, `DateTime` and many other modules that will help you handle time conversions. In our case, we can use an example:

    use Timex

    expire_date = Date.now |> Date.add(Time.to_timestamp(7, :days))
    secs_to_expire_date = Date.now |> Date.diff(date_to, :secs)

    conn |> put_resp_cookie("first_cookie_key", "first_cookie_value", max_age: secs_to_expire_date)

Contributors: [Hubert Łępicki](mailto:hubert.lepicki@amberbit.com) [Rafał Maksymczuk](mailto:rafal.maksymczuk@amberbit.com) [Wojciech Piekutowski](mailto:wojciech.piekutowski@amberbit.com)

