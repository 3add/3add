#This is an advanced auction house skript made by 3add.
#Please report bugs.
#This is a custom auction house skript I made to work with some skript addons.
#The idea is pretty simple. You use "/ah" to view the auction house listings.
#You can use "/ahlist <price>" to list an item/item stack you're holding on the auction house.
#It's made with a custom inventory interface with a confirm purchase screen.
#There are a couple known bugs but nothing major. It needs some quality of life updates as well.
#This is a fully functioning beta test build.



command /ah [<text>] [<integer>]:
	trigger:
		if arg-1 is not set:
			displayAH(0,player)
			set {%player%_ah_page} to 0
		else:
			if arg-1 is "check":
				if player has permission "ah.admin":
					loop {AuctionHouse::items::*}:
						message "%loop-index%: %{AuctionHouse::items::%loop-index%}%" to player
				else:
					send "&cNo permission"
			if arg-1 is "credites":
				send "" and "&6Credites: 3add and _NotAdam_" and ""
			if arg-1 is "sell" or "list":
				if arg-2 is not set:
					send "&b/ah sell price &3<-- is the correct usage."

				if arg-2 > 0:
				
					if tool is not air:
						set {_listingFee} to floor(arg 2 * 0.05)
						if {deposited.coins::%uuid of player%} >= {_listingFee}:
							if {numberOfListings::%player%} is not set:
								set {_size} to size of {AuctionHouse::items::*}

								set {AuctionHouse::items::%{_size}%} to held item
								set {AuctionHouse::owner::%{_size}%} to player
								set {AuctionHouse::price::%{_size}%} to arg 2

								set held item to air
								set {numberOfListings::%player%} to 1
								subtract {_listingFee} from {deposited.coins::%uuid of player%}
								message "&6Item listed. Listing fee: &e$%{_listingFee}%" to player

								refreshListings()
							else if {numberOfListings::%player%} < 5:
								set {_size} to size of {AuctionHouse::items::*}

								set {AuctionHouse::items::%{_size}%} to held item
								set {AuctionHouse::owner::%{_size}%} to player
								set {AuctionHouse::price::%{_size}%} to arg 2

								set held item to air
								add 1 to {numberOfListings::%player%}
								subtract {_listingFee} from {deposited.coins::%uuid of player%}
								message "&6Item listed. Listing fee: &e$%{_listingFee}%" to player
	
								refreshListings()
							else:
								message "&cYou have too many listings! &e(Max 5)" to player
						else:
							message "&cYou dont have enough money to list at that price. &e($%{_listingFee}%)" to player
					
					else:
						message "&cYou can't list air on the auction house.." to player
				else:
					send "&3&lERROR | &bYour selling price must be above 0"
			if arg-1 is "clear":
				if player is not set:
					set {AuctionHouse::olditems::*} to {AuctionHouse::items::*} and {AuctionHouse::olditems::*}
					set {AuctionHouse::oldowner::*} to {AuctionHouse::owner::*} and {AuctionHouse::oldowner::*}
					set {AuctionHouse::oldprice::*} to {AuctionHouse::price::*} and {AuctionHouse::oldprice::*}
					delete {AuctionHouse::items::*}
					delete {AuctionHouse::owner::*}
					delete {AuctionHouse::price::*}
					delete {numberOfListings::*}
					set {aucresettimer} to {orgtimer}
					send "&cCleared the auction house &4THIS CANNOT BE UNDONE"
					broadcast "" and "&bThe aucion house got cleared view your old listings at &3Old Listings" and ""
				if player has permission "ah.admin":
					set {AuctionHouse::olditems::*} to {AuctionHouse::items::*} and {AuctionHouse::olditems::*}
					set {AuctionHouse::oldowner::*} to {AuctionHouse::owner::*} and {AuctionHouse::oldowner::*}
					set {AuctionHouse::oldprice::*} to {AuctionHouse::price::*} and {AuctionHouse::oldprice::*}
					delete {AuctionHouse::items::*}
					delete {AuctionHouse::owner::*}
					delete {AuctionHouse::price::*}
					delete {numberOfListings::*}
					set {aucresettimer} to {orgtimer}
					send "&cCleared the auction house &4THIS CANNOT BE UNDONE"
					broadcast "" and "&bThe aucion house got cleared view your old listings at &3Old Listings" and ""
				else:
					send "&cNo permission"
			if arg-1 is "open":
				make player say "/ah"
			if arg-1 is "help":
				send "" and "&b&l      ACION HOUSE HELP      " and "" and "&b/ah [open] &3--> used to open the aucion house." and "&b/ah sell/list &3--> Used to sell items on the auction house." and "&b/ah check &3--> used to check all the items on the auction house. &cONLY FOR OPS" and "&b/ah clear &3--> Used to clear all the items on the auction house. &cONLY FOR OPS" and "&b/ah help &3--> sends you the help messages" and "&b/ah credites &3--> sends you the credites." and "&b/ah settimer &3--> change the acion house reset timer (in minutes)" and "&b/ah claim &3--> check if you have items to claim in old listings" and "" and "&6Credits: 3add and _NotAdam_" and ""
			if arg-1 is "settimer":
				if player has permission "ah.admin":
					if arg-1 is set:
						set {aucresettimer} to "%arg-2% minutes" parsed as timespan
						set {orgtimer} to "%arg-2% minutes" parsed as timespan
						send "&bChanged the acion house timer to &3%arg-2% minutes"
					else:
						send "&bthe acion house timer is &3%arg-2% minutes"
				
			if arg-1 is "claim":

				loop {AuctionHouse::oldowner::*}:
					if {AuctionHouse::oldowner::%loop-index%} contains player:
						send "" and "&bYou still have unclaimed items from previus aucion houses /ah to claim!" and ""

						exit
				if size of {AuctionHouse::oldowner::*} is 0:
					send "" and "&3You do not have any items to claim!" and ""
					delete {items.%player%}
					
				
                
every second:
	remove 1 second from {aucresettimer}
	if {aucresettimer} > 0 seconds:
	else:
		
		execute console command "ah clear"
	



on inventory close:
	if {%player%_viewingAH} is true:
		set {%player%_viewingAH} to false
	if {%player%_viewingListings} is true:
		set {%player%_viewingListings} to false

on inventory click:
	if name of event-inventory is "&5Auction House":
		cancel event
		wait 1 tick
		if event-slot is not air:
			if line 5 of lore of event-slot is set:
				set {_listing} to line 5 of lore of event-slot
				set {_listing} to subtext of {_listing} from characters 12 to the length of {_listing} parsed as integer

				if {AuctionHouse::owner::%{_listing}%} is not event-player:

					if {AuctionHouse::price::%{_listing}%} <= {deposited.coins::%uuid of player%}:
						displayPurchase({_listing},player)
					else:
						message "&cYou don't have enough money to purchase that!" to event-player

				else:
					message "&cYou cant purchase your own listing!" to event-player
			else:
				if event-slot is paper named "&2Next Page":
					add 1 to {%player%_ah_page}
					displayAH({%player%_ah_page},player)
				else if event-slot is paper named "&4Previous Page":
					subtract 1 from {%player%_ah_page}
					displayAH({%player%_ah_page},player)
				else if event-slot is gold ingot named "&6My Listings":
					displayMyListings(player)
				else if event-slot is gold ingot named "&6My old Listings":
					displayMyOldListings(player)
				else if event-slot is oak sign named "&6Search":
					close player's inventory
					send "" and "&bType your search:" and ""
					set {searching.%player%} to true



	else if name of event-inventory is "&6Purchase Item - &eAre you sure?":
		cancel event
		set {_listing} to line 5 of lore of slot 4 of open inventory
		set {_listing} to subtext of {_listing} from characters 12 to the length of {_listing} parsed as integer
		if event-slot is green stained glass pane:
			set {_item} to slot 4 of player's open inventory
			if "%{AuctionHouse::items::%{_listing}%}%" is "%{_item}%":
				if player has enough space for {AuctionHouse::items::%{_listing}%}:
					subtract {AuctionHouse::price::%{_listing}%} from {deposited.coins::%uuid of event-player%}
					add {AuctionHouse::price::%{_listing}%} to {deposited.coins::%uuid of {AuctionHouse::owner::%{_listing}%}%}
					send "&bYour item &3%{AuctionHouse::items::%{_listing}%}% &bwas soled to &3%player% &bfor &3%{AuctionHouse::price::%{_listing}%}%" to {AuctionHouse::owner::%{_listing}%}
					give event-player {AuctionHouse::items::%{_listing}%}
					message "&eItem purchased" to event-player
					if {AuctionHouse::owner::%{_listing}%} is not "Server":
						if {AuctionHouse::owner::%{_listing}%} is online:
							message "&2Item listing &e%{AuctionHouse::items::%{_listing}%}% &2has sold for &e$%{AuctionHouse::price::%{_listing}%}%" to {AuctionHouse::owner::%{_listing}%}
						else:
							add "&2Item listing &e%{AuctionHouse::items::%{_listing}%}% &2has sold for &e$%{AuctionHouse::price::%{_listing}%}%" to {%{AuctionHouse::owner::%{_listing}%}%::messages::*}
							subtract 1 from {numberOfListings::%{AuctionHouse::owner::%{_listing}%}%}
						if {_listing} + 1 < size of {AuctionHouse::items::*}:
							delete {AuctionHouse::items::%{_listing}%}
							delete {AuctionHouse::owner::%{_listing}%}
							delete {AuctionHouse::price::%{_listing}%}
							fixListings()
							displayAH({%player%_ah_page},player)
							refreshListings()
						else:
							delete {AuctionHouse::items::%{_listing}%}
							delete {AuctionHouse::owner::%{_listing}%}
							delete {AuctionHouse::price::%{_listing}%}
							displayAH({%player%_ah_page},player)
							refreshListings()
					else:
						displayAH({%player%_ah_page},player)
						message "&2Item listing &e%{AuctionHouse::items::%{_listing}%}% &2has sold for &e$%{AuctionHouse::price::%{_listing}%}%" to console
				else:
					message "&cYou don't have room!" to player
			else:
				displayAH({%player%_ah_page},player)
				message "&cListing mismatch. Item may have been sold already." to player

		else if event-slot is red stained glass pane:
			displayAH({%player%_ah_page},player)

	else if name of event-inventory is "&6My Listings":
		cancel event
		if line 5 of lore of event-slot is set:
			set {_listing} to line 5 of lore of event-slot
			set {_listing} to subtext of {_listing} from characters 12 to the length of {_listing} parsed as integer
			if player has enough space for {AuctionHouse::items::%{_listing}%}:
				give event-player {AuctionHouse::items::%{_listing}%}
				message "&eListing removed" to event-player
				subtract 1 from {numberOfListings::%player%}
				if {_listing} + 1 < size of {AuctionHouse::items::*}:
					delete {AuctionHouse::items::%{_listing}%}
					delete {AuctionHouse::owner::%{_listing}%}
					delete {AuctionHouse::price::%{_listing}%}
					fixListings()
					displayAH({%player%_ah_page},player)
					refreshListings()
				else:
					delete {AuctionHouse::items::%{_listing}%}
					delete {AuctionHouse::owner::%{_listing}%}
					delete {AuctionHouse::price::%{_listing}%}
					displayAH({%player%_ah_page},player)
					refreshListings()
			else:
				message "&cYou don't have room!" to player
		else if event-slot is red wool block named "&4Back":
			displayAH({%player%_ah_page},player)
	else if name of event-inventory is "&6My old Listings":

		cancel event
		if line 5 of lore of event-slot is set:
			set {_listing} to line 5 of lore of event-slot
			set {_listing} to subtext of {_listing} from characters 12 to the length of {_listing} parsed as integer
			if player has enough space for {AuctionHouse::olditems::%{_listing}%}:
				give event-player {AuctionHouse::olditems::%{_listing}%}
				message "&eSuccesfully Claimed" to event-player
				if {_listing} + 1 < size of {AuctionHouse::olditems::*}:
					delete {AuctionHouse::olditems::%{_listing}%}
					delete {AuctionHouse::oldowner::%{_listing}%}
					delete {AuctionHouse::oldprice::%{_listing}%}
					displayAH({%player%_ah_page},player)
				else:
					delete {AuctionHouse::olditems::%{_listing}%}
					delete {AuctionHouse::oldowner::%{_listing}%}
					delete {AuctionHouse::oldprice::%{_listing}%}
					displayAH({%player%_ah_page},player)
			else:
				message "&cYou don't have room!" to player
		else if event-slot is red wool block named "&4Back":
			displayAH({%player%_ah_page},player)
function displayAH(page: integer, p: player):
	show chest inventory with 6 rows named "&5Auction House" to {_p}
	set {%{_p}%_viewingAH} to true
	set {_slot} to 0

	loop 45 times:
		set {_index} to {_page} * 45
		add {_slot} to {_index}
		if {AuctionHouse::items::%{_index}%} is set:
			set {_item} to {AuctionHouse::items::%{_index}%}
			set line 1 of lore of {_item} to "&8--------------------"
			set line 2 of lore of {_item} to "&eOwner: &a%{AuctionHouse::owner::%{_index}%}%"
			set line 3 of lore of {_item} to "&ePrice: &a$%{AuctionHouse::price::%{_index}%}%"
			set line 4 of lore of {_item} to "&8--------------------"
			set line 5 of lore of {_item} to "&8Listing ##%{_index}%"
			set slot {_slot} of {_p}'s open inventory to {_item}

			set {_slot} to {_slot} +1

	if {_page} > 0:
		set slot 45 of {_p}'s open inventory to paper named "&4Previous Page"
	else:
		set slot 45 of {_p}'s open inventory to black stained glass named " "

	set slot 46 of {_p}'s open inventory to gold ingot named "&6My Listings"
	set slot 47 of {_p}'s open inventory to black stained glass named " "
	set slot 48 of {_p}'s open inventory to sunflower named "&bNext ah reset: &3%{aucresettimer}%"
	set slot 49 of {_p}'s open inventory to black stained glass named " "
	set slot 50 of {_p}'s open inventory to oak sign named "&6Search" with lore "&7Search through the items."
	set slot 51 of {_p}'s open inventory to black stained glass named " "
	set slot 52 of {_p}'s open inventory to gold ingot named "&6My old Listings"
	set slot 53 of {_p}'s open inventory to black stained glass named " "

	if size of {AuctionHouse::items::*} > (({_page} + 1) * 45):
		set slot 53 of {_p}'s open inventory to paper named "&2Next Page"
	else:
		set slot 53 of {_p}'s open inventory to black stained glass named " "


function displayPurchase(listing: object, p: player):
	set {_item} to {AuctionHouse::items::%{_listing}%}
	set line 1 of lore of {_item} to "&8--------------------"
	set line 2 of lore of {_item} to "&eOwner: &a%{AuctionHouse::owner::%{_listing}%}%"
	set line 3 of lore of {_item} to "&ePrice: &a$%{AuctionHouse::price::%{_listing}%}%"
	set line 4 of lore of {_item} to "&8--------------------"
	set line 5 of lore of {_item} to "&8Listing ##%{_listing}%"

	show chest inventory with 1 row named "&6Purchase Item - &eAre you sure?" to {_p}
	set slot 0 of {_p}'s open inventory to green stained glass pane named "&2Confirm"
	set slot 1 of {_p}'s open inventory to green stained glass pane named "&2Confirm"
	set slot 2 of {_p}'s open inventory to green stained glass pane named "&2Confirm"
	set slot 3 of {_p}'s open inventory to green stained glass pane named "&2Confirm"
	set slot 4 of {_p}'s open inventory to {_item}
	set slot 5 of {_p}'s open inventory to red stained glass pane named "&4Cancel"
	set slot 6 of {_p}'s open inventory to red stained glass pane named "&4Cancel"
	set slot 7 of {_p}'s open inventory to red stained glass pane named "&4Cancel"
	set slot 8 of {_p}'s open inventory to red stained glass pane named "&4Cancel"

function fixListings():
	set {_loopnumber} to 0
	loop {AuctionHouse::items::*}:
		set {AuctionHouse::items::%{_loopnumber}%} to {AuctionHouse::items::%loop-index%}
		add 1 to {_loopnumber}
	set {_size} to size of {AuctionHouse::items::*} - 1
	delete {AuctionHouse::items::%{_size}%}

	set {_loopnumber} to 0
	loop {AuctionHouse::owner::*}:
		set {AuctionHouse::owner::%{_loopnumber}%} to {AuctionHouse::owner::%loop-index%}
		add 1 to {_loopnumber}
	set {_size} to size of {AuctionHouse::owner::*} - 1
	delete {AuctionHouse::owner::%{_size}%}

	set {_loopnumber} to 0
	loop {AuctionHouse::price::*}:
		set {AuctionHouse::price::%{_loopnumber}%} to {AuctionHouse::price::%loop-index%}
		add 1 to {_loopnumber}
	set {_size} to size of {AuctionHouse::price::*} - 1
	delete {AuctionHouse::price::%{_size}%}

function displayMyListings(p: player):
	show chest inventory with 6 rows named "&6My Listings" to {_p}
	set {%{_p}%_viewingListings} to true
	set {_slot} to 0

	loop {AuctionHouse::owner::*}:
		if loop-value is {_p}:
			set {_item} to {AuctionHouse::items::%loop-index%}
			set line 1 of lore of {_item} to "&8--------------------"
			set line 2 of lore of {_item} to "&eOwner: &a%{AuctionHouse::owner::%loop-index%}%"
			set line 3 of lore of {_item} to "&ePrice: &a$%{AuctionHouse::price::%loop-index%}%"
			set line 4 of lore of {_item} to "&8--------------------"
			set line 5 of lore of {_item} to "&8Listing ##%loop-index%"
			set slot {_slot} of {_p}'s open inventory to {_item} named "&4Left click to remove listing"

			set {_slot} to {_slot} +1
	set slot 53 of {_p}'s open inventory to red wool block named "&4Back"
function displayMyOldListings(p: player):	
	show chest inventory with 6 rows named "&6My Old Listings" to {_p}
	set {%{_p}%_viewingListings} to true
	set {_slot} to 0

	loop {AuctionHouse::oldowner::*}:
		if loop-value is {_p}:
			set {_item} to {AuctionHouse::olditems::%loop-index%}
			set line 1 of lore of {_item} to "&8--------------------"
			set line 2 of lore of {_item} to "&eOwner: &a%{AuctionHouse::oldowner::%loop-index%}%"
			set line 3 of lore of {_item} to "&ePrice: &a$%{AuctionHouse::oldprice::%loop-index%}%"
			set line 4 of lore of {_item} to "&8--------------------"
			set line 5 of lore of {_item} to "&8Listing ##%loop-index%"
			set slot {_slot} of {_p}'s open inventory to {_item} named "&4Click to claim"

			set {_slot} to {_slot} +1
	set slot 53 of {_p}'s open inventory to red wool block named "&4Back"

every 30 minutes:
	loop all players:
		loop {AuctionHouse::oldowner::*}:
			{AuctionHouse::oldowner::%loop-index%} contains loop-player
			
			send "" and "&bYou still have unclaimed items from previus aucion houses /ah to claim!" and "" to loop-player
			exit
on chat:
	if {searching.%player%} is true:
		cancel event 
		set {_gui} to chest inventory with 6 rows named "&bSearching:&3 %message%"
		set {_slot} to 0
		loop {AuctionHouse::items::*}:
			if "%type of loop-value%" contains "%message%":
				set {found.%player%} to true
				set {_item} to loop-value
				set line 1 of lore of {_item} to "&8--------------------"
				set line 2 of lore of {_item} to "&eOwner: &a%{AuctionHouse::owner::%loop-index%}%"
				set line 3 of lore of {_item} to "&ePrice: &a$%{AuctionHouse::price::%loop-index%}%"
				set line 4 of lore of {_item} to "&8--------------------"
				set line 5 of lore of {_item} to "&8Listing ##%loop-index%"
				set slot {_slot} of {_gui} to {_item} named "&4Click to buy"

				add 1 to {_slot} 
		if {found.%player%} is true:
			delete {found.%player%}
		else:
			set slot 0 of {_gui} to redstone named "&cDid not find an item called ""&4%message%&c"""
		set slot 53 of {_gui} to barrier named "&cGo Back"
		delete {searching.%player%}
		open {_gui} to player
on inventory click:
	if name of event-inventory contains "&bSearching:&3 ":
		cancel event
		if name of event-item contains "&cGo Back":
			make player say "/ah"
		if line 5 of lore of event-slot is set:
			set {_listing} to line 5 of lore of event-slot
			set {_listing} to subtext of {_listing} from characters 12 to the length of {_listing} parsed as integer

			if {AuctionHouse::owner::%{_listing}%} is not event-player:

				if {AuctionHouse::price::%{_listing}%} <= {deposited.coins::%uuid of player%}:
					displayPurchase({_listing},player)
				else:
					message "&cYou don't have enough money to purchase that!" to event-player

			else:
				message "&cYou cant purchase your own listing!" to event-player
	
				
function refreshListings():
	loop all players:
		if {%loop-player%_viewingAH} is true:
			displayAH({%loop-player%_ah_page},loop-player)
		else if {%loop-player%_viewingListings} is true:
			displayMyListings(loop-player)
on tab complete of "/ah":
	set tab completion for position 1 to "sell" and "list" and "clear" and "open" and "check" and "help" and "credites" and "settimer" and "claim"
	set tab completion for position 2 to "<number>"
