The trailing controller can be found in PP.py, which is the python file responsible for pure pursuit for the car.

Trailing controller itself has the following description:
```
Adjust the speed of the ego car to trail the opponent at a fixed distance

        Inputs:

            speed_command: velocity of global raceline

            self.opponent: frenet s position and vs velocity of opponent

            self.position_in_map_frenet: frenet s position and vs veloctz of ego car

        Returns:

            trailing_command: reference velocity for trailing
```

The module gets 3 inputs and returns only 1 output.

## Inputs:
* speed_command

	speed_command is set to 0 in line **76** in the initialization of the PP file and is later given a value based on the [[calc_speed_command]] Function which calculates it based on the car's velocity vector and the lateral normalized error.

* self.opponent

	frenet s position and vs velocity of opponent

* self.position_in_map_frenet

	frenet s position and vs veloctz of ego car

**Ego car** is just referring to where the car THINKS it is, so the car calculates its speed to trail an opponent and all the calculations is based on the position of the ego car, naturally carefully monitoring and ensuring minimal error in these coordinates is of high importance.

## Outputs:
* trailing_command

	reference velocity for trailing

# What it actually does / inner working

The function itself does a couple of stuff, it

1. calculates the gap between the car and the opponent
2. stores the value of the calculation in an intermediate variable 
3. compares the 2 values and names it error
4. compares the velocity between the car and the opponent
5. TBC
