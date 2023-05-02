# sqlalchemy-challenge
DataViz SQLAlchemy Challenge

Part 1 Jupiter Notebook and analysis accessible through SurfsUp > Climate Notebook

Part 2 API app accessible through SurfsUp > app.py, code also listed below

# Import the dependencies.
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func
import datetime as dt
import numpy as np
from flask import Flask, jsonify


#################################################
# Database Setup
#################################################
engine = create_engine("sqlite:///Resources/hawaii.sqlite")

# reflect an existing database into a new model
Base = automap_base()

# reflect the tables
Base.prepare(autoload_with=engine)

# Save references to each table
Measurements = Base.classes.measurement
Station = Base.classes.station

# Create our session (link) from Python to the DB
# session = Session(engine)

#################################################
# Flask Setup
#################################################
app = Flask(__name__)

#################################################
# Flask Routes
#################################################

@app.route("/")
def welcome():
    return (
        f"Welcome to the Sam's Surfs Up API!<br/>"
        f"Available Routes:<br/>"
        f"/api/v1.0/precipitation<br/>"
        f"/api/v1.0/stations<br/>"
        f"/api/v1.0/tobs<br/>"
        f"/api/v1.0/insert(yyyy-mm-dd)<start><br/>"
        f"/api/v1.0/insert(yyyy-mm-dd)<start>/(yyyy-mm-dd)<end>"
    )

@app.route("/api/v1.0/precipitation")
def precipitation():
    # Create our session (link) from Python to the DB
    session = Session(engine)

    # Query all results
    results_precipitation = session.query(Measurements.date,Measurements.prcp).\
                                filter(Measurements.date >= '2016-08-23').\
                                order_by(Measurements.date).all()

    session.close()

    # Create a dictionary from the row data and append to a list of preciitation
    all_precipitation = []
    for date, prcp in results_precipitation:
        precip_dict = {}
        precip_dict["date"] = date
        precip_dict["prcp"] = prcp
        all_precipitation.append(precip_dict)


    return jsonify(all_precipitation)

@app.route("/api/v1.0/stations")
def stations():
    # Create our session (link) from Python to the DB
    session = Session(engine)

    # Query all stations
    station_results = session.query(Station.station).all()

    session.close()

    # Convert list of tuples into normal list
    all_stations = list(np.ravel(station_results))

    return jsonify(all_stations)

@app.route("/api/v1.0/tobs")
def temp_monthly():

    session = Session(engine)

    # Query the primary station for all tobs from the last year
    # From our previous notebook we know that 'USC00519281' is the most active station
    
    # Calculate the date 1 year ago from last date in database
    temp_results = session.query(Measurements.tobs).\
                    filter(Measurements.date >= '2016-08-23').\
                    filter(Measurements.station == 'USC00519281').all()

    
    session.close()

    # Unravel results into a 1D array and convert to a list
    # Convert list of tuples into normal list
    all_temps = list(np.ravel(temp_results))
    
    # Return the results
    return jsonify(all_temps)

# Return a JSON list of the minimum temperature, the average temperature, and the maximum temperature for a specified start or start-end range.
# For a specified start, calculate TMIN, TAVG, and TMAX for all the dates greater than or equal to the start date.
# For a specified start date and end date, calculate TMIN, TAVG, and TMAX for the dates from the start date to the end date, inclusive.

@app.route("/api/v1.0/insert(<start>)")
def temp_range_start(start):
    session = Session(engine)
    temp_range = session.query(func.min(Measurements.tobs), func.avg(Measurements.tobs), func.max(Measurements.tobs)).\
        filter(Measurements.date >= start).all()
    session.close()

    temp_range_list = []
    for min,avg,max in temp_range:
        tob_dict = {}
        tob_dict["Min"] = min
        tob_dict["Average"] = avg
        tob_dict["Max"] = max
        temp_range_list.append(tob_dict)

    return jsonify(temp_range_list)

@app.route("/api/v1.0/insert(<start>)/(<stop>)")
def temp_range_start_stop(start=None,stop=None):
    session = Session(engine)
    temp_range = session.query(func.min(Measurements.tobs), func.avg(Measurements.tobs), func.max(Measurements.tobs)).\
        filter(Measurements.date >= start).filter(Measurements.date <= stop).all()
    session.close()

    temp_range_list = []
    for min,avg,max in temp_range:
        tob_dict = {}
        tob_dict["Min"] = min
        tob_dict["Average"] = avg
        tob_dict["Max"] = max
        temp_range_list.append(tob_dict)

    return jsonify(temp_range_list)

if __name__ == "__main__":
     app.run(debug=True)