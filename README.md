# CS3700 Project 2: Router
Group Members: Sean Hoerl & Jess Van de Ven

## Developer Notes

### High-Level Approach
Use a forwarding table to track the destination networks/ports. 
Entries on this forwarding table were aggregated when consecutive binary representation network/netmask entries had the same dest port. 

We handled incoming message of different types in the following ways:
    Update: add message to our update cache, update the forwarding table, forward the update, aggregate the forwarding table as needed
    Data: send message 
    Dump: send message
    Withdraw: add mesage to our withdrawal cache, update the forwarding table, forward the withdraw


### Challenges Faced
The logic for handling customers on the source or destination end of a message took a bit to understand the difference between. The logic for aggregating also was particular and took iterating before it worked as expected under every case. 


### Properties/Features
When removing an entry from the forwarding table, if there are no entries with the given removal network/netmask representation, the entire table needs to be rebuilt. To rebuild, everything from our update message cache is used to recreate the table, and everything from our withdrawal cache is removed to edit the table for accuracy. Then the new table is aggregated as needed.


### Testing
Using the tests from the test suite. Analyzing the output from running the router with periodic print statements to identify router or forwarding table interactions.
