# Other stuff

We have observations for trips and stop, and we've matched the former to the road network, but the data isn't quite ready, and there's a bunch of housekeeping.

## Residuals{-}

Reading about trips and stops you may have realised that if we only record a trip and stop when the stop finishes we'll usually have some data left over from a trip stop pair that hasn't finished yet. These are the residuals, and we can fairly easily stick them on to the front of the next batch of data. The problem arises because we want to link chains of stops (more about that later in the example work). Linking stops in order in a batch is easy enough, but between batches is a bit trickier. We could append an id of the last stop to the residual data but that's a bit dangerous. This is why the first observation of a trip is also the last observation of the stop, we not only get the true start of a trip, but we can link the unique time stamps across batches.

## Time and geocoding{-}

So far we have only cared about relative time for a given vehicle; which observations come before and after and by how much. As such we could have all the times in Universal Mean Time, or in a Unix epoch; that is the number of seconds since Midnight on January 1st 1970. This is also important because if we always used local time, trucks might travel suddenly forward or backwards in time when they cross state borders and time zones.

But we care a great deal about the local time in our analysis; knowing the times of day when rest stops and road segments are most used or congested. To do this we need to know what time zone each stop and observation is in. In the Australian context that means knowing what state they are in.

This is good for other reasons. Later in our analysis we will care a lot about not only about which state something happened, but smaller units of Australian geography such as SA2s. If we attribute a small unit like SA2 to our trips and observations we get the time zone, and we also get a quick spatial index for our later work.

Finding out what area points are in is a well known problem, but with our volumes of data it can take a lot of tim matching hundreds of millions of observations. Luckily almost all of these are already matched to less than 2 million road segments which tend to stay in one state, so all we need to do is map those segments and the stops.

There's a problem however. We are using the Australian Statistical Geographical Standard (ASGS) polygons from the ABS, and they hug the coastline very, very closely and sometimes hug the banks of major waterways near the coast. This means that the Bradfield Highway for instance, which used the Sydney Harbour Bridge, is not in any SA2 or Australia State, at least according to these polygons. Not are any number of vehicular ferries, such as the Spirit of Tasmania. In addition, coastlines change. I don't just mean erosion and sediment and tides. People reclaim land and build on it. In particular they do this near ports where land is scarce and valuable, and we have a lot of trucks stopping at ports. We can manually change the shapefiles when this happens, but we'd have to be vigiliant. 

The solution is pretty simple - when we fail to match an SA2 we just match to the nearest one instead - but this was a good example of an unexpected problem that needed solving.

## Locations{-}

Whilst our stop definition doesn't need known locations, there are specific locations we care about, in particular rest areas. If we're matching stops to to SA2s, we may as well match them to rest areas as well, which we can get from other sources. May as well do loading zones where we have data for that as well. And wait, every street address in the country is part [G-NAF](https://data.gov.au/dataset/ds-dga-19432f89-dc3a-4ef3-b943-5326ef1dbecc/details) so we can match to that as well. Since these are all known before processing we can use an algorithm like rtree to quickly find which location is closest and, if it is close *enough* determine our stop was at that location.

## Confidentiality and clustering{-}

Lastly, we care a lot about the specific location (rather than general, SA2 level location) of some stops, but not others. If a truck stops at a port, or a rest stop, or an informal rest area, that is very important to the project and a reason why firms provided their data in the first place. But stops at particular businesses are less important and, if published, might betray commercial information. Worse still, some drivers take their trucks home, so their stops can expose their place of residence. We need a way of distinguishing places we are interested in from places that are none of our business.

For locations like rest stop and ports that's easy, but what about informal rest areas? Our solution was to use a clustering algorithm. It finds find where a large number of stops were together in time and space, such as a regularly used patch of dirt by a highway. We can then filter these stops based on how may vehicles. If it is only one vehicle it might be someone's home, and will be ignored. If it is only one firm it might be a client or a depot, so we will also ignore it. But if it is used by a variety of firms it is of interest to us.

We've got a lot of information now, so how to we handle it?
