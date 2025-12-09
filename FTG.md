FTG Stands for Follow The Gap which works by receiving the LiDAR array for distances, and turning it into a steering angle and speed

The file has initialization variables for the different speed option for the code to choose from depending on the situation.

## `_preprocess_lidar`

```
Preprocess the LiDAR scan array.

  

        This method performs preprocessing on the LiDAR scan array. The preprocessing steps include:

        1. Setting each value to the mean over a specified window.

        2. Rejecting high values (e.g., values greater than 3m).


        Parameters:

            ranges (numpy.ndarray): The LiDAR scan array.  

        Returns:

            numpy.ndarray: The preprocessed LiDAR scan array.
```

This function sets the radians per element in the array of the LiDAR as to know which index corresponds to what angle. This value is stored in the variable `self.radians_per_elem`

Then, an array is set from the 360 degrees, as we are not going to use the full array, only the parts that's infront of us, this range is set to `-135 to 135` degrees

The function returns an array of the LiDAR elements from -135 to 135 degrees in order, and clipped between 0 and the max lidar distance.

## `_get_steer_angle`

```
Get the angle of a particular element in the LiDAR data and

        transform it into an appropriate steering angle.

        Parameters:

            point_x (float): The x-coordinate of the LiDAR data point

            point_y (float): The y-coordinate of the LiDAR data point

        Returns:

            float: The transformed steering angle
```

as the name entails, this function is a simple math equation to return the angle of the best point chosen.

## `_get_best_range_point`

```
Find the best point i.e. the middle of the largest gap within the bubble radius.

        Parameters:

            proc_ranges (list): List of processed ranges.

        Returns:

            tuple: The x and y coordinates of the best point.
```

The function starts by getting the Largest gap indexes, and returning the `x` and `y` cartesian points to follow, which is the middle of the gap that's been identified and chosen to follow.

## `_find_largest_gap`

```
Find the index of the starting and ending of the largest gap and its width

        Parameters:

            ranges (numpy.ndarray): Array of range values
            radius (float): Threshold radius value
  
        Returns:

            tuple: A tuple containing the index of the starting of the largest gap, the index of the ending of the largest gap, and the width of the largest gap.
```

This function binarizes the array of the LiDAR elements, it goes through each element and checks if it passes the threshold for it to be considered a 'gap', this comparison is now saved in a new array called `bin_ranges`

### Gemini explanation:
This line creates a new NumPy array called `bin_ranges`. It performs a **boolean comparison** element-wise on the input array `ranges` against the scalar `radius`:

- If an element in `ranges` is **greater than or equal to** `radius` ($\ge$), the corresponding element in `bin_ranges` is set to **1** (or `True`).
    
- If an element in `ranges` is **less than** `radius` ($<$), the corresponding element in `bin_ranges` is set to **0** (or `False`).

The function also has an array for identifying the transition between `No Gap` to `Gap`, by using 
`diff( )` we have an array that's always 1 when there's a transition and a 0 when the current state continues, this is used to know the boundaries of the gap

Then it iterates through the array to know the biggest gap possible to follow, and returns the index of the `Left boundary` and the `Right boundary`. 

## `_safety_border`

```
Add a safety bubble if there is a big increase in the range between two points.
  
        Parameters:
            ranges (list): List of range values.

        Returns:
            np.ndarray: Array of filtered range values.
```

this function checks the distance between each array element and the next, to place the safety bubble at the edges of the objects to not place them in our FTG decisions, we only choose the gap that's not super close to edges to avoid our car hitting the walls or scraping against them, this is done by two `For` loops that goes once in the positive direction and once in the other direction to cover the entire area of the LiDAR scan.

## `_get_radius`

```
        Calculate the radius based on the track width and velocity.

        Returns:
            float: The calculated radius.
```

simply calculates the radius of the safety bubble according to the track width

## `process_lidar`

```
        Process each LiDAR scan as per the Follow Gap algorithm &
        calculate the speed and steering angle.

        Parameters:
            ranges (list): List of LiDAR scan ranges

        Returns:
            tuple: A tuple containing the speed and steering angle
```

This function utilizes all the functions into one main function.