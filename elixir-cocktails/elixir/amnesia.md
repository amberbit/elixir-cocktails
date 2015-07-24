#The problem

Suppose you have to use a simple database in an elixir application. How to do that?

##The solution
Use Amnesia, a simple wrapper of Erlang's mnesia database.

##Adding Amnesia to dependencies
To add Amnesia, edit your mix.exs and add a new dependency in function

    defp deps do
        [{:amnesia, github: "meh/amnesia", tag: :master}]
    end

After that, run

    mix deps.get

to download Amnesia

##Creating the database
Let's create a database for battery status – so there will be a timestamp, a status – (e.g. “Charging”, “Discharging”) and a percentage. To use the database, we first have to create a file that will be used to create the database.


To create database, add a new file “Database.ex” in your lib folder. Write it as follows:

    #Firstly, we have to require Amnesia to use defdatabase and deftable
    require Amnesia
    use Amnesia
    
    defdatabase Database do
    	#We define a table, records will be sorted, the first element will be taken as an index
		deftable Battery, [:timestamp, :status, :percentage ], type: :ordered_set do 
			#Nice to have, we declare a struct that represents a record in the database
			@type t :: %Battery{timestamp: non_neg_integer, status: String.t, percentage: non_neg_integer}
		end
    end
Look out – it's not just “String”, it's “String.t”

If you want to use another index, just add it as an option in deftable, e.g.
    
    deftable Battery, [:timestamp, :status, :percentage ], index: [:status], type: :ordered_set do 


To install the database, run 

    iex -S mix 

in the terminal. We chose the disk mode of the database – the records will be saved on the disk. There is another option – database in RAM.
The best option for installing the database – use Mix Tasks. For this example, we will write it in iex.

    Interactive Elixir (1.0.5) - press Ctrl+C to exit (type h() ENTER for help)
    iex(1)> require Amnesia
    nil
    iex(2)> use Amnesia
    nil
    iex(3)> Amnesia.Schema.create
    :ok
    iex(4)> Amnesia.start
    :ok
    iex(5)> Database.create(disk: [node]) # or for RAM – Database.create(ram: [node])
    [:ok, :ok]
    iex(6)> Database.wait
    :ok

Just remember – run it once, otherwise there will be errors like “:already_exists”!

#Reading and writing

Reading from and writing to the database requires Amnesia.transaction. If you don't use it, there will be errors like

    ** (exit) {:aborted, :no_transaction}

##Writing to the database

Let's add some random values to the database:

    iex(1)> use Database
    [nil, nil, nil, nil, nil, nil]
    iex(2)> Amnesia.transaction do
    ...(2)> %Battery{timestamp: 0, percentage: 67, status: "Charging"} |> Battery.write
    ...(2)> r = %Battery{timestamp: 12, percentage: 75, status: "Discharging"} 
    ...(2)> r |> Battery.write 
    ...(2)> # is equal to
    ...(2)> # Battery.write(r)
    ...(2)> end
    %Database.Battery{percentage: 67, status: "Charging", timestamp: 0}
    %Database.Battery{percentage: 75, status: "Discharging", timestamp: 12}

##Reading from the database

You can use read – its looks up a record with an index equal to the param (in this case it looks for the timestamp)

    iex(1)> use Database
    [nil, nil, nil, nil, nil, nil]
    iex(2)> Amnesia.transaction do
    ...(2)> Battery.read(0)       
    ...(2)> end
    %Database.Battery{percentage: 67, status: "Charging", timestamp: 0}

If there is no such a record with this index, the result is nil

    iex(3)> Amnesia.transaction do
    ...(3)> Battery.read(2)
    ...(3)> end
    nil

If we want to find records that meet the conditions (e.g. where the percentage is in range (50..70) and we only want to know the status of the records (excluding the timestamp and the percentage)). 
When we found the records, we have to invoke the method Amnesia.Selection.values with param r – our query. Next we use IO.inspect on every record from the query.
If we use “where”, we use the standard Elixir syntax, so words like “or” and “and”. 

    iex(3)> Amnesia.transaction do
    ...(3)> r = Battery.where percentage >= 50 and percentage <= 70, select: [status]
    ...(3)> r |> Amnesia.Selection.values |> Enum.each &IO.inspect(&1)
    ...(3)> # is equal to
    ...(3)> # Enum.each(Amnesia.Selection.values(r), fn(x) -> IO.inspect(x) end) 
    ...(3)> end
    ["Charging"]
    :ok

##Uninstalling the database
When we want to destroy the database (for example, a user wants to uninstall our app), we use:

    iex(1)> Amnesia.start
    :ok
    iex(2)> Database.destroy
    [:ok, :ok]
    iex(3)> Amnesia.stop
    :stopped
    iex(4)> 
    13:59:42.791 [info]  Application mnesia exited: :stopped
    Amnesia.Schema.destroy
    :ok






