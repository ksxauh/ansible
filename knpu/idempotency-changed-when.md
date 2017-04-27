# Idempotency changed when

I run my playbook with just the deploy tag. This is what it looks like right now. Notice there are several tasks that say, "Changed", even though that's not really true. First, you notice that the two tasks around composer, download composer and move composer. These reporting as changed. That's something we're gonna work on later. Right now I want to focus down here on our last four tasks. Fixing the directory permissions and setting up an execute migrations. Now why is this change to versus just okay? Meaning not changed important. Well, remember that our tasks are meant to be item potent which means that it should be safe to run a task over and over and over again without having any side effects.

Actually our tasks are item potent right now. If we run this fix VAR directory permissions task over and over and over again, it's not gonna change every single time we do it, it's only gonna set the directory the first time and then afterwards, nothing should change. The issue is that our task is not reporting that. It thinks that it's changing every time, when really it's not changing. Now, right now that doesn't really seems like an important detail, but later we're gonna talk [inaudible 01:15] actually making decisions in our playbook based on whether or not something changed.

In fact, we've already seen this one time with our handlers. We found out that when we set up a handler. Notify it, restart engine X, the handler's only notified when that task changed. As a best practice, as much as we can, we want our tasks to correctly report whether or not they changed. The fix VAR directory permissions one, that one is a little surprising that that one isn't reporting correctly. Reason built in file module and we're telling it to change the mode to seven times seven and recurse yes. It's because of that recursion that it's always coming back as changed true.

In this case, we really know that this task is item potent. It's safe to run over and over again without side effects. The easiest way to tell this, that it's not changed is to say "changed_when", and set that to "false". Now when we're on the playbook. This time it will report that as not changed. Perfect. What about these other ones down here? For example, creating the database. Technically, the first time we run this, that is creating the database and then every time after that, it's not creating the database. If we really want this task to be smart, we should actually be able to detect whether or not it did or did not need to create the database.

There's actually a really cool way to do that. First add a register key under it and set that to "db_create_result". What that's doing is it's going to set the output of this command to a new variable called, db create result. This is called a fact because we're collecting facts about our system. To see what that looks like, below this we can temporarily set up a debug task and we're gonna print db create results. The debug module, and I'm using the shorthand here, is a really cool module that helps you print out variables to see what they look like.

Below this, I'm also gonna put this in the deploy tag so that we can just run that to see the result. Alright, let's try it. Back over, run our playbook. Whoa, awesome. Check this out. We have this awesome thing where we can see the command that was run, we can see when it was started, and most importantly, we can see the standard out, what printed out when we run that command. This is database symfony for connection and default already exists skipped. This is really cool because we can actually use that language to detect whether or not this needed to change. I'm actually gonna copy that already exists skipped part, head over here, and remove my debug task and now up here we can use changed when, but we can use it in a much more intelligent way.

Instead of just saying false like we did before, we can set up a big expression. Watch this, we'll say, not {{ and we'll use db_create create_result which is the variable that we just registered before, dot standard out (.stout) }}. We're using standard out because that's the key on the variable that we want to use. We can then pipe that through a search filter. That's a special filter inside of [ginga 06:47] and look for inside single quotes, already exists, skipped. End the single quote, end the parentheses, and the two curly, and the double quote.

If you're used to using [twig 07:00], this is gonna look very, very familiar. This is our variable, reading [inaudible 07:03] variable, we're piping it through a filter. In ginga there's a filter called, search and that's gonna return true or false and then we're negating that upfront. Down here for execute a migration, we can do something very, very similar. First, we're gonna register a new variable called, db migrations results and below that I'm gonna copy the changed when and pace that. What we need here is we need to figure out what language happens when there are no migrations to execute. If you flip over to your virtual machine, make sure you're inside your project directory, and we'll run bin/console doctrine:migrations:migrate--no--interaction.

You see when it prints out when there's nothing to migrate it says, no migrations to execute. Let's copy that language. Same thing, we'll go over here and paste that in to our little expression and make sure that you're using the new variable, db migration results. Awesome. Last one is loading the data fixtures. This one's a little tricky because technically, this runs on every single deploy and it actually fully empties the database and re-adds the fixture. In some ways, you can say this is always changing because it's always emptying the data fixtures.

Another way you can say is it's never changing because it's just guaranteeing that the database is in a certain state. For now, I'm just gonna set this to changed when false. It probably doesn't matter and it only really matters if we start taking conditional actions based on whether or not this did something. For now, I'll use changed when false so that it looks like it wasn't changed. Okay. With all this, let's flip back over, go to our host machine, and rerun the playbook.

Got it. You can see now, all green down here. Not changed. But if we did something totally unreasonable on our ... instead of a virtual machine like doctrine:database:drop--force, this time we need those to show up as changed. Because it will need to create the database and now the database will be empty so it will need to run the migrations. Beautiful. Changed and changed on the database and migrations.
