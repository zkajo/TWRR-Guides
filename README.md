# TWRR-Guides
 
### **What is "Scripting"**

Scripting is a way to perform certain actions which are to occur under certain conditions, by writing code. In simplest terms, we can specify what we want the game to do, when we want it to do it, and our commands will be executed. Scripts can do things that game mechanics normally do not cater for. Complicated scripts can even introduce completely new mechanics to the game.

This guide assumes the user knows nothing about scripting, so we'll start with the basics. If you're an advanced user or know a little bit about programming, just pick and choose the bits to read.

### **Limitations**

Controlling the UI is very limited. We are able to open windows and show buttons via script, but we have no way of producing new UI windows with new data, meaning you'll have to get creative about displaying your mechanics info to the user. A lot of stuff can be done through unique buildings and character traits and ancilliaries: it will be up to you to figure this stuff out. Number of commands available in scripting is quite big, but there will always be things that are missing. It would take far too long list all possibilities and limitations here, but rest assured that with a bit of creativity you'll be able to do a lot of really unique things. Most commands are documented under: `Feral Interactive\Total War ROME REMASTERED\VFS\Local\Rome\documentation`

### **Tools**

Scripts are just text files. Windows `Notepad` will do (or equivilent on your system since some of you really don't like Windows and are sending me mental vibes that you are indeed on another, better system ;) ), although I'd recommend something a little more robust. Personally I like `notepad++`, but anything like `sublime`, or `visual studio code` will do. Really. Anything. If you're a new modder, really, do yourself a favour and install something other than Notepad. You'll never look back and will save yourself literally HOURS of time.

### **Where can scripts be used**

Normally we run the main script `every time the campaign starts`. There are other instances when scripts can be run:

- Major event triggers (to activate and deactivate them)
- Ancilliaries and Trait triggers
- Spawn scripts for emergent / dead factions
- On advisor click/call
- Probably some others that I don't remember now, but will eventually list here

### **Campaign Script**

Probably the most versatile and important script which runs throughout the campaign. You can call it in `descr_strat`, by writing the following lines at the end of the file:

```
script
your_script.txt
```

`your_script` can be called whatever you wish, as long as it sits in the same folder as `descr_strat`, and has the `.txt` extension.

The script always starts with `script` command, and ends with `end_script`. Any lines that begin with `;;` are ignored. A simple example would be this:

```
script
    ;; your other code goes here
    ;; comments are good for writing explanatory messages in your code when you're
    ;; performing some black magic which you'll forget how it works later
end_script
```

It should be noted that the campaign script is ALWAYS ran when the campaign starts. If you have ANY setup (such as variable delcarations), you should do it all wrapped up in an if statement on turn zero. For example:

```
script
    
    if I_TurnNumber = 0
        ;;your setup goes here, this will be called only ONCE per whole campaign, because we only have one turn 0
        ;;it could be potentailly called more than once if the player saves and reloads the game on turn 0
        ;;to safe guard against this, you could use a persistent counter discussed later

        ;;if I_TurnNumber = 1 would not work, because we wouldn't get to turn 1 before 
        ;;this script stopped executing UNLESS we saved, and reloaded on turn 1
    end_if

;;your monitors go here. They will be tracked by the game all the time
end_script
```



## **Script Components**

### **Monitor Events**

Monitor events are what it says on the tin: an event which monitors conditions and fires up when those conditions are met. Let's look at a simple monitor event which we can put into our campaign script. The whole thing would look like this:

```
script

	monitor_event FactionTurnStart FactionType romans_julii
	and I_TurnNumber < 30
		console_command add_money romans_julii, 1000
	end_monitor

end_script
```

Let's look at individual components:

`monitor_event` is a declaration of the event.
`FactionTurnStart` is the event being monitored.
`FactionType romans_julii` is our first condition, meaning this event will only be monitored for House of Julii faction.

Our second condition is `and I_TurnNumber < 30` which means the turn number has to be lower than 30 for this event to occur.

`console_command add_money romans_julii, 1000` is a simple command to give Julii 1000 denari.

`end_monitor` ends this command block.

In simple English, it's this: Every time Julii start a turn, AND the turn number is lower than 30, give Julii 1000 denari.
Each event provides a "scope". This scope can be, for example, a character, a settlement or a faction. Certain conditions will not work in certain scopes. For example, it's not possible to check whether a faction is a faction leader, because it's the wrong scope: this condition only works for characters.

You have to be careful your use of conditions and commands, as sometimes you can provide a wrong scope. On a good day, it will simply ignore it. On a bad day, it can cause a crash.

### **Condition Events**

We can also monitor certain conditions, rather than events. Consider this example:

```
;;do something on turn 30
monitor_conditions I_TurnNumber = 30
    ;;add some code for your logic here
    terminate_monitor ;;dont track it anymore, because why? It won't be called anyway...
end_monitor
```

### **If statements and conditions**

Another way to control additional conditions is by using `if` statements. They are an additional command block which can be inserted into code, which allows us to test for a specific condition. Consider this code:

```
script

	monitor_event FactionTurnStart FactionType romans_julii
	and I_TurnNumber < 30
		console_command add_money romans_julii, 1000
	
        if not FactionIsAlive gaul
            console_command add_money romans_julii, 1000
        end_if
    end_monitor

end_script
```

Presuming we enter the monitor event as usual, now the Julii will receive additional 1000 denari `IF gaul faction is NOT alive`. Remember that this if statement exists INSIDE the event, and will only be called if the conditions of the event itself are met. Please note that `not` is a keyword you can add to conditions!

Consider the following example:
```
script

	monitor_event FactionTurnStart FactionType romans_julii
	and not FactionIsLocal
	and (
        I_TurnNumber < 30
	    || Treasury < 5000
    )
		console_command add_money romans_julii, 1000
	
        if not FactionIsAlive gaul
            console_command add_money romans_julii, 1000
        end_if
    end_monitor

end_script
```

Here the conditions are as follows: Faction has to be The Julii, AND it cannot be local faction (player), AND (the turn number has to be under 30 OR the treasury be under 5000) denari. Note the brackets - these conditions are evaluated together, just like in simple mathematics, and because we have OR ( || ), only one of these has to be true. Positioning of the brackets is also important. It might not look pretty, but it works, so keep it the same!

Your if statements can be as complicated as you wish.

### **Random Chance**

You can add random chance to conditions by using `RandomPercent` condition. For example:

```
script

	monitor_event FactionTurnStart FactionType romans_julii
	and RandomPercent > 90	
        console_command add_money romans_julii, 1000
    end_monitor

end_script
```

The above script will trigger with the usual condition AND has 10% chance of firing.  RandomPercent, as name suggests is between 0 and 100. Let's have a look at use of RandomPercent in total conquest. For context, Persia doesn't start with any settlement.

```
	monitor_event SettlementTurnStart   ;;we monitor the settlement on its turn start. Scope is Settlement
    and I_LocalFaction persia           ;;if the player is playing as persia
    and	SettlementOwnedBy seleucid      ;;and the settlement is owned by seleucid
    and	not IsCapital                   ;;and it's not a capital city
    and HasResource local_persia        ;;and has a local hidden resource called local_persia
    and RandomPercent > 80              ;;and generated number is bigger than 80 (20% chance) 
        provoke_rebellion local	        ;;provoke rebellion in THIS settlement
        terminate_monitor               ;;kill this event so it does not happen again
	end_monitor
```

It should be noted that this script works in conjunction with another. By itself, a revolt will simply be a revolt, but because we have a spawn script, Persia will actually spawn here.

### **Counters**

`Counters`, often called `variables` are tokens which we can increment. They have a unique name and value assigned to them. They can be local, global or persistent.

- `local` counters are only used in that particular scope and will be reinitialised every time we enter the scope again.
- `global` counters are those which are used throughout the campaign script but are reset when we restart the game (probably not useful anymore, but were very important in OG).
- `persistent` counters are new in Remastered and have their value attached to the save file, meaning you can use track them throughout the campaign.

How can we use counters? Well, it's completely up to you and the mechanics you want to implement. You can track player's progress, fire up some events or commands depending on when the counter reaches certain threshold.

You can declare counters like this:

- persistent: `declare_persistent_counter name_of_your_counter`
- global and local: `declare_counter name_of_your_counter`

You can set their value like this:

- `set_counter name_of_your_counter X` where X is the value

You can increment and decrement their value like this:

- `inc_counter name_of_your_counter X` where X is the value. For decrementing, just use the minus sign in front of the number

You can check counter value like this:

- `if I_CompareCounter counter_name OPERATOR X` where `OPERATOR` is a mathematical function like =, >, < etc. and X is the value

Example use of counters. Let's take a look at how we can use counters to trigger something cool, like the Roman Civil War!

```
script
    ;;because these counters are persitent, they are tied to the save file, meaning we can keep tracking them on reload!
    declare_persistent_counter romans_julii_support
    declare_persistent_counter romans_brutii_support
    declare_persistent_counter romans_scipii_support

    ;;increment them when someone with a corresponding trait becomes faction leader
    monitor_event CeasedFactionHeir FactionType romans_julii
	and IsFactionLeader
		if Trait House_Brutii = 1
			inc_counter romans_brutii_support 50
		end_if
		if Trait House_Scipii = 1
			inc_counter romans_scipii_support 50
		end_if
		if Trait House_Julii = 1
			inc_counter romans_julii_support 50
		end_if		
	end_monitor

    ;;decrement whenever general loses battle
	monitor_event PostBattle FactionType romans_julii
	and FactionIsLocal
    and IsGeneral
    and not WonBattle
		if Trait House_Brutii = 1
			inc_counter romans_brutii_support -5
		end_if
		if Trait House_Scipii = 1
			inc_counter romans_scipii_support -5
		end_if
		if Trait House_Julii = 1
			inc_counter romans_julii_support -5
		end_if
	end_monitor

    ;;if support for one of the houses reaches 300
	monitor_event FactionTurnStart FactionType romans_julii
	&& (           
        I_CompareCounter romans_julii_support > 300
        || I_CompareCounter romans_brutii_support > 300
        || I_CompareCounter romans_scipii_support > 300
    )
        ;;some code here that will contain logic for triggering civil war
        terminate_monitor ;;terminate it because we only want the war to trigger once
    end_monitor

end_script
```


### **Local scope**

One of the biggest scripting limitations of OG Rome was lack of local scope in commands and conditions. We could only refer to settlements, characters or factions by calling their explicit id, meaning that if you wanted to call a command which takes in a coped name, like add_population, you'd have to make a different monitor event for every possible id in the game. To illustrate, the example above takes in "local" scope, meaning the revolt will occur in the settlement the monitor event is currently refering to. In OG, we'd have to make if statement for every single settlement, to make sure we're calling the right command.

You DO have to be careful when using local scope however. You don't want to use a "isGeneral" on a settlement for example, because bad things can happen. Let's have a look at some other cool stuff you can do with local scope that were not possible before! This script is a "major event" trigger.

```
script
	declare_counter owns_italy  ;;declare a local counter
	set_counter owns_italy 1    
	
	if I_CompareCounter italian_conquest > 0    ;;if value of italian_conquest (declared in campaign script) is zero return false
		return false
	end_if
	
	for_each settlement in world                           ;;go through every settlement in the world, changing scope each time
        if HasResource local_italy                         ;;if this settlement has hidden resource local_italy
		&& not I_SettlementOwner local = romans_julii      ;;if local settlement is owned by the Julii
			set_counter owns_italy 0            
		end_if		
	end_for
	
	if I_CompareCounter owns_italy > 0
		return true
	end_if

	return false
end_script
```

This script iterates through ALL italian settlements (we know they're italian because they have local_italy resource). If romans_julii control all of these provinces, the event triggers. In OG we'd have to write an if statement FOR EVERY SETTLEMENT EXPLICITLY, making this potentially a few hundred lines of code.

### **Loops**

We have access to `for each` loops, and `while loops`. While loops call code as long as the conditions of the loop are met. For example, they can add 1000 denari to your purse every tick, providing you with unlimited gold (and possibly crashing, I don't know :) ). While loops have some uses, but they should be used with care. They were very important in OG as they kept the campaign script running.

You can declare while loops like this:

```
while your_condition_here
    ;;your code goes here
end_while
```

for each loops are very useful as they allow you to iterate through a list, and provide you with a local scope. The following two examples will work on turn end.
```
if I_LocalFaction romans_julii
    for_each settlement in faction "romans_julii"
        provoke_rebellion local ;;ha ha, die julii, die
    end_for
end_if
```

This will work as well:
```
    for_each settlement in faction "local"
            provoke_rebellion local
    end_for
``'