# CS3700 Project 2: Router
Group Members: Sean Hoerl & Jess Van de Ven

## Developer Notes

### High-Level Approach
Use a forwarding table to track the destination networks/ports (while also storing all of the correspoding info like netmask, ASPath, etc...). Store all update and withdrawal messages recieved for later use when rebuilding. Entries on this forwarding table were aggregated when consecutive binary representation network/netmask entries had the same dest port. 

We handled incoming message of different types in the following ways:
* Update:
    *  add message to our update cache 
    *  update the forwarding table,
    *  forward the update to all other addresses in our forwarding table (except one we recived it from), attempt to aggregate the forwarding table
* Data: 
    * find the best match in our forwarding table for the destination address
    * send message to that address
* Dump: 
    * send copy of our forwarding table to destination address
* Withdraw: 
    * add mesage to our withdrawal cache
    * update the forwarding table (disaggregate if needed)
    * forward the withdraw

To find the best match in our forwarding table for sending data messages, we used bitwise logic to ensure an entry was actually valid for a given destination ip. Since there can be multiple valid entries, we used the logic described in the project description to break ties. For aggregation, we go through each entry in our forwarding table and calculate what the neighboring entry would be using the network and netmask, and then see if that calculated entry exists in our forwarding table (and also that it has all of the same corresponding info like netmask, ASPath, and such). If we do have a match, then we know the lower address of the two entries will be the address of our new aggregated entry, and then we correspondingly update the netmask by shifting it left one. We then remove the two entries which can be aggregated from our forwarding table, and add our new aggregated entry to the forwarding table.

When we see a withdrawal message and need to remove an entry from our forwarding table, we remove all entries from our forwarding table that have the corresponding network, netmask, and peer (which is source ip of withdrawal message). Since we know that all entries in a withdrawal message must have been in our forwarding table at some point, if the length of our forwarding table is the same after we try to remove an entry from the withdrawal message, we know that the entry must have been aggregated, thus we will need to disaggregate. To disaggregate we throw away the forwarding table, add all entires from update cache, remove all entries from withdrawal cache, and then aggregate as many times as needed to ensure all entries which need to be aggregated are.

### Challenges Faced
The logic for handling customers on the source or destination end of a message took a bit to understand the difference between. The logic for aggregating also was particular and took iterating before it worked as expected under every case. Figuring out the bitwise logic for finding a neighboring network was particularly difficult, and we found it more straight forward to just use the string form of the binary representation of each int in the address/netmask, and then make functions to iterate through those. While not particularly elegant, we thought that there would be a lot of overhead associated with changing the ip address string into its 32 bit int form just to be able to use bitwise operators there that weren't particularly readable (though we thought they were readable for the case of determining if a entry was valid for forwarding data messages). The logic for determining a better entry between two was also a bit challenging and long-winded, but the documentation provided in the project description was helpful for figuring this out.

### Properties/Features
When removing an entry from the forwarding table, if there are no entries with the given removal network/netmask representation, the entire table needs to be rebuilt. To rebuild, everything from our update message cache is used to recreate the table, and everything from our withdrawal cache is removed to edit the table for accuracy. Then the new table is aggregated as needed. We think that we did a good job of splitting things into functions and finding ways to reuse said functions. This helped improved the readability of our code. We also think aggregating as many times as needed after rebuilding our forwarding table from scratch was particularly thoughtful, as we would be able to catch the case where we have a forwarding table with many aggregated entires, and then need to disaggregate one of them, but still want the rest of them to be aggregated (rather than only aggregating one entry after rebuilding our table)

### Testing
Using the tests from the test suite. Analyzing the output from running the router with periodic print statements to identify router or forwarding table interactions. We also were able to leverage print statements so that we could see exactly what our code was doing through our different iterations of development. When figuring out all of the logic for finding best matches or neighboring addresses, we would draw out the binary representations of everything and figured out exactly what we wanted to do, and then made our functions to correspond with/represent that. For some of our more utility based functions (changing addresses into arrays of ints or binary strings, etc..), we were able to pull them out of this project and test them in different cases in a different file / online compilers, allowing us to ensure that those functions were doing exactly what we wanted them to do. Designing our router iteratively (corresponding with different development steps as outlined in project spec) helped us keep things relatively simple at each stage and abstract out things which we already knew we had working properly. This seemed to work well for our group.
