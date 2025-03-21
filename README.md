# AI API

To cite this software, use:

```bib
@ARTICLE{10681400,
  author={Lelis, Claudio A. S. and Roncal, Julio J. and Silveira, Leonardo and De Aquino, Roberto Douglas G. and Marcondes, Cesar A. C. and Marques, Johnny and Loubach, Denis S. and Verri, Filipe A. N. and Curtis, Vitor V. and De Souza, Diego G.},
  journal={IEEE Access},
  title={Drone-Based AI System for Wildfire Monitoring and Risk Prediction},
  year={2024},
  volume={12},
  number={},
  pages={139865-139882},
  keywords={Wildfires;Predictive models;Sensors;Normalized difference vegetation index;Measurement;Drones;Artificial intelligence;Environmental monitoring;Machine learning;Risk management;Spatiotemporal phenomena;Aerial drones;artificial intelligence;environmental monitoring;machine learning;risk assessment;spatiotemporal data;wildfire detection;wildfire risk estimation},
  doi={10.1109/ACCESS.2024.3462436}}
```

## API Use:

### End-Points:

#### Prediction end-point `\prediction`:

- Prediction end-point: `/prediction`
- POST request.
- End-point request body:

1. `"lat_begin": float or null`
2. `"lat_end": float or null`
3. `"long_begin": float or null`
4. `"long_end": float or null`
5. `"time_begin": string or null`, format 'YYYY-MM-DD HH:MM:SS'
6. `"time_end": string or null`, format 'YYYY-MM-DD HH:MM:SS'
7. `"tag": string`. The `tag` should be one of the options:
`humidity`, `pressure`, `temperature`, `VARI`, `vNDVI`.

Request body example:

```
{
    "lat_begin": -23.210,
    "lat_end": -23.200,
    "long_begin": -45.898,
    "long_end": -45.887,
    "time_begin": "2024-03-09 16:05:23",
    "time_end": "2024-03-10 02:23:39",
    "tag": "humidity"
}
```

- Important:

1. The only key that cannot be `null` or `None` is the `tag` key.
In case the other keys are not provided the default behavior is to use the data boundaries as the upper and lower limits for prediction.
2. The `time_begin` and `time_end`, if provided, will be use for filtering the data, but not during prediction.
3. Prediction will be only performed if these is at least 100 data points available for the provided interval requested.

- Prediction default settings:

1. Number of samples: 70
2. Probability (bootstrapping): 0.85
3. Number of neighbors: 10
4. Size of mesh for grid discretization: 100

- Request return:

The request will return a dictionary containing 4 lists of data.
Each list of lenght `(size-of-mesh * size-of-mesh)`. For the default parameters `(100*100)`.

The lists are: `prediction`, `confidence`, `latitude`, `longitude`.

An example of how to use the output of the request to plot the prediction and uncertainty heat-maps can be seen at the file `data_analysis.py`, in the function `request_prediction`.

#### Get data statistics end-point `/get_data_stats/<tag>`:

- Get data stats end-point: `/get_data_stas/<tag>`
- GET request.

- Returns basic statistics about the data of a given tag.

- The `<tag>` parameter can take on the values:
`all`, `humidity`, `pressure`, `temperature`, `VARI`, `vNDVI`.

- Example of response, for tag `VARI`:

```
{
    "entries": "4297",
    "lat_max": -23.157380713316538,
    "lat_min": -23.163075023211118,
    "long_max": -45.789681888149374,
    "long_min": -45.79763483464151,
    "time_max": "2024-04-20 20:53:17",
    "time_min": "2023-11-28 19:24:00",
    "value_max": 0.1663888888888889,
    "value_mean": 0.0011936784061231226,
    "value_min": 0.0,
    "value_std": 0.004603846809462936}
```

- Obs: `entries` is the number of records of that `tag` in the database. The other values should be self-explanatory.

#### Tags info end-point `/tags_info`:

- Tags info end-point: `/tags_info`
- GET request.

- The purpose of this end-point is to query the database and return the number of records present for each tag.

- Example of response:

```
{
    "VARI": 4199,
    "humidity": 2,
    "pressure": 1,
    "string": 1,
    "temperature": 2,
    "vNDVI": 4199
}
```

#### Get clean data end-point `/get_clean_data/<tag>`:

- Tags info end-point: `/get_clean_data/<tag>`
- GET request.

- Returns all the data queried from the database, after removing duplicates.

- The `<tag>` parameter can take on the values:
`all`, `humidity`, `pressure`, `temperature`, `VARI`, `vNDVI`.

- The return dictionary is formed by 4 lists, with the values of:
`latitude`, `longitude`, `value`, `timestamp`.

## Run locally using Docker:

- Build the application: `docker compose build`
- Run the application expose pon port 8000: `docker run -p 8000:8000 ai_api_wildfire:1.0`
- Make a request to: `http://localhost:8000/some_endpoint`

## Run on production

1. Commit any changes
2. Run Action `Build Docker Image CI`
3. Run Action `Helm Deploy to production`

ATTENTION: the deployment to production (action 3) is attached to the latest commit hash,
so if you want to deploy to production after some commit you musht first build a new image, or the deployment
will be attached to a hash that may not be associated with any image. This results in an error where the kubernetes pod cannot
download its corresponding image.

## Interaction with larger system

![system_diagram](system_diagram.png)

The AI API is placed in the middle of the system:

- It receives a request from the front-end;
- It makes the same request to collect the data to the DB-API;
- It generates the prediction and confidence maps;
- It sends the prediction and confidence maps to the front-end.
