# Handling URL redirects in Phoenix with Plug

## The problem

You are migrating your legacy Ruby on Rails web application (;>) to a brand new,
shiny, [Elixir](http://elixir-lang.org) + [Phoenix](http://www.phoenixframework.com) version. URL structure of the old and new
application mostly matches, but some of the pages changed their
location. You need to keep the old URLs redirecting to new URLs because
your visitors might have them bookarked. Google has also all your
current URLs indexed, so it would be nice to point it to new locations
when it crawls your web site next time.

## The solution

Create a custom [Plug](http://hexdocs.pm/plug) and use it in your
endpoint.ex before your application's router even kicks in.

First, you need to create a plug and place it under `lib/` directory of
your Phoenix application. Mine is called
`ElixirCocktailUrlRedirects.RedirectsPlug` and I placed it under
`lib/elixir_cocktail_url_redirects/redirects_plug.ex`:

    defmodule ElixirCocktailUrlRedirects.RedirectsPlug do
      import Plug.Conn

      def init(options), do: options

      def call(conn, options) do
        to = options[full_path(conn)]
        do_redirect(conn, to)
      end

      defp do_redirect(conn, nil), do: conn

      defp do_redirect(conn, to) do
        conn
          |> Phoenix.Controller.redirect(to: to)
          |> halt
      end
    end

And I needed to mount my plug in
`lib/elixir_cocktail_url_redirects/endpoint.ex` with:

    plug ElixirCocktailUrlRedirects.RedirectsPlug, %{"/" => "/page"}

In my case I am simply redirecting root URL to `/page`, but you can
specify multiple values in a map, such as:

    plug ElixirCocktailUrlRedirects.RedirectsPlug, %{"/old/" => "/new/", "/old2/" => "/new2/"}

For a complete reference, see [this commit on
Github](https://github.com/amberbit/elixir_cocktail_url_redirects/commit/be5b8082613930f0c75a3762ad1bf9e28a7a0436)

## Extra tips

Your use case might be a bit more complicated. You might have a need to
redirect by regexp or implement more complex redirect logic. Check out
[Plug.Conn reference](http://hexdocs.pm/plug/Plug.Conn.html) and inspect
list of plugs in your `endpoint.ex` to find a solution to your problem.
You should be able to access URL parameters, session data, cookies and
other information from your plug, if you really need it.

If your redirects will be permanent, make sure your plug [is setting HTTP
301 status code, instead of the default
302](https://github.com/amberbit/elixir_cocktail_url_redirects/commit/a76736d4a8dbb555abe3f8688a3eb06dbe5faad1).

Contributors:
[Hubert Łępicki](mailto:hubert.lepicki@amberbit.com)
[@hubertlepicki](http://twitter.com/hubertlepicki)


