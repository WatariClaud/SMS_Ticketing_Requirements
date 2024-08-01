## Getting STarted:

### _BACKEND_

 - ### _Models_
 
 
 	
 	User: {
 	
 		id: number,
 		
 		name: string,
 		
 		mobile_number: string,
 		
 		is_engineer: boolean #manages role on frontend views,
 		
 		ref_number: number #track users referred by another user,
 		
 		referred_by: number `#User[id]` of referrer
 		
	}
	
	
	Admin: {
	
		id: number,
		
		name: string,
		
		email: string,
		
		password: string
		
	}
	
	
	Station: {
	
		id: number,
		
		managed_by: string, #human_resource.id
		
		is_open: boolean
		
	}
	
	
	Human_Resource: {
	
		id: number,
		name: string,
		email: string,
		assigned_to_station: number #station.id
	}
	
	
	Ticket: {
	
		id: number,
		by_user: number,
		start_time: datetime,
		end_time: date_time,
		total_waiting_time: number, #integer storing total seconds spent waiting
		number_of_activities: number,
		served_by: number #human_resource.id
	}
	
	
	Activity: {
	
		id: number,
		
		by_ticket: number #ticket.id,
		
		is_waiting: boolean,
		
		completed_ boolean,
		
		cancelled: boolean,
		
		status: string,
		
		created_on: datetime,
		
		closed_on: datetime		
	}
	
	
	Notification: {
	
		id: number,
		
		referred: number #user.id,
		
		referrer: number #user.id,
		
		created_on: datetime,	
	}
	
	
	### Background

	`Ticket[total_waiting_time]` is the cumulative time in hours and minutes spent by a suser waiting at different stations.
	
	When a user is assigned to a station and before they leave the station, if they are being served, that tracks an 
	
	*Activity (is_waiting: false)*
	
	When they are waiting to be served, that tracks an activity as well (is_waiting: true).
	
	When closing a ticket (end_time), calculate all activities for this ticket.
	
	 - Use the `Activity[created_on]` and `Activity[closed_on]` to calculate the time taken to complete an Activity.
	 
	 ---
	
	### <u>Use Case</u>
	
	When a ticket is opened, it automatically creates an Activity as follows:
	 - If having to wait for a station, `Activity[is_waiting]` is true.
	 
	 	When moving from `Activity[is_waiting]=true` to `Activity[is_waiting]=false`, close Activity and create new Activity
	 	
	 - If immediately assigned to a station, Activity[is_waiting] is false, no need to create a new Activity
	 
	When user is switching stations:
	
	 - `Activity[completed]` is true
	 
	 - Create new Activity for next station
	 
	 - If having to wait, same as opening a new ticket above (new `Activity[is_waiting]=true`)
	 
	 - If not having to wait, use Activity until next Activity
	 
	Repeat until user closes ticket, log all Activities for this ticket and associated<br>
	`Ticket[Activity[created_on]]` and `Ticket[Activity[closed_on]]` to get all time spent on this ticket.
	
	Closing an Activity should automatically open a new one unless explicitly closing the ticket.
	
	If no next Station assigned, new `Activity[is_waiting]` is true.
	
	If `Activity[is_waiting]=true` is completed, add the `"Activity[closed_on]` - `Activity[created_on]"` to current `Ticket[total_waiting_time]`
	
	**May consider an edge case where a user leaves the site without closing the ticket**
	
	In the above edge case, the next assigned `Station[managed_by]` can mark the ticket as closed, automatically closing the current Activity within (n) minutes and perform all calulations as usual.
	
	
	---
	
	### Relationships
	- A User can have multiple Tickets.
	
	- A Ticket can have multiple Activities.
	
	- A HumanResource manages a Station.
	
	- A Station is managed by a HumanResource.
	
	---
	
	### Required

	When a user is referred by another user, send notification on dashboard and an SMS to both users (referred and referrer)
	
	Send an SMS to the user with, and avail to dashboard view:
	
	<code>
	Message: {
		username: User[name],
		total_stations_viisited: Activity[by_ticket]=Ticket[id] and Activity[is_waiting]=false,
		total_time_spent_on_site: Ticket[closed_on] - Ticket[opened_on]
		total_waiting_time
	}
	</code>
	
