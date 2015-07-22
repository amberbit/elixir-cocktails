# Handling URL redirects

## The problem

You are migrating your legacy Rails application (;>) to a brand new,
shiny, Elixir + Phoenix version. URL structure of the old and new
application mostly matches, but some of the pages changed their
location. You need to keep the old URLs redirecting to new URLs because
your visitors might have them bookarked. Google has also all your
current URLs indexed, so it would be nice to point it to new locations
when it crawls your web site next time.

## Solution

Create a custom [Plug](http://hexdocs.pm/plug) and use it in your
endpoint.ex before your application's router even kicks in.

First, you need to create a plug and place it under `lib/` directory of
your Phoenix application. Mine is called
`ElixirCocktailUrlRedirects.RedirectsPlug` and I placed it under
`lib/elixir_cocktail_url_redirects/redirects_plug.ex`:

{: .language-elixir .prettyprint }
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


{: .language-elixir .prettyprint }
  plug ElixirCocktailUrlRedirects.RedirectsPlug, %{"/" => "/page"}

Contributors:
[Hubert Łępicki](mailto:hubert.lepicki@amberbit.com)
[@hubertlepicki](http://twitter.com/hubertlepicki)

