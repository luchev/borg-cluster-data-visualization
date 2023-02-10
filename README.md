# Borg Data Visualization

A visualization aiming to help exploring Google's [cluster manager Borg's dataset](https://github.com/google/cluster-data)

<img width="1161" alt="image" src="https://user-images.githubusercontent.com/4147570/211326719-fe5242ca-72a6-4674-989a-d16a8ec2acc6.png">

The visualization includes:
* Filters by date (the date is mocked for readibility)
* A TreeMap with resources per job/machine/instance
* A Tree Graph to help navigate the parent-child relationships between jobs
* An event timeline to help explore events for a selected job/machine/instance

## How to create a subset of the data locally so one can work with it?

We need to query one of the tables of the Borg's data using Big Query.

We used this set of collection IDs (replace them wherever you see `...` in the queries bellow).

```
376806789282, 377785058171, 377788307885, 377798680268, 381105511426, 382447880122, 383195444727, 383608861807, 383935133305, 384115980813, 384348004759, 384424169653, 384616480023, 385521821265, 393963753334, 394100656170, 395001670373, 395309057390, 396114412610, 397188062864, 397267075458, 398651117810, 398966051786, 399735044474, 399932045779, 400083630480, 400133700345, 400359412340, 400359504871, 400424637904, 400424409030, 384348000733, 384115980665, 396094938307, 399653678562, 400017778662, 397187982822, 400285450623, 381105483183, 397267071150, 398966042563, 385521796700, 377785052963, 395309053288, 384616436814, 377798675385, 394100645117, 395001641360, 383935116162, 382447868413, 377788304537, 398651109881, 376806742728, 400133630643, 382428282200, 398648504325, 319956351863, 397263856681, 395001437952, 383884105961, 377787092035, 377797560396, 398963569490, 377782527181, 384254552569, 381057460613, 394097188469, 385521585571, 384603189188, 395307610239, 384603189188, 394097188469, 385521585571, 395307610239, 319956351863, 377787092035, 383884105961, 398961750192, 397263856681, 398963569490, 395001437952, 395304012910, 377797560396, 384254552569, 377782527181, 381057460613, 382428282199, 398648504325
```

Querying tables, which have a collection id (e.g. `clusterdata_2019_a.collection_events`):
```
SELECT *
FROM `google.com:google-cluster-data.clusterdata_2019_a.collection_events`
WHERE collection_id IN (...)
```

Querying tables, which have a machine id column (e.g. `clusterdata_2019_a.machine_attributes`):
```
SELECT * FROM `google.com:google-cluster-data.clusterdata_2019_a.machine_attributes` WHERE machine_id IN
  (
    SELECT DISTINCT(machine_id)
    FROM `google.com:google-cluster-data.clusterdata_2019_a.instance_events`
    WHERE collection_id IN (...) 

    UNION

    DISTINCT SELECT machine_id
    FROM `google.com:google-cluster-data.clusterdata_2019_a.instance_usage`
    WHERE collection_id IN (...) 
  )
```

The tables `machine_events` and `machine_attributes` have `machine_id` as their primary key and mainly have Machine data
The tables `collection_events`, `instance_usage`, and `instance_events` have `collection_id` as their primary key.

Download the queries data from Big Query as JSON entries.

Finally, run the `dataparsing/fill.py` script to preprocess the JSON files into SQL as well as some basic preprocessing of the data. Read the [dataprocessing README](https://github.com/luchev/borg-cluster-data-visualization/blob/main/dataparsing/README.md) for more details.
