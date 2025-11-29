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
3. compares the values of the actual gap calculated in the steps above, to the value that it SHOULD be, and that's the gap error.
4. compares the velocity between the car and the opponent and that's the velocity error
5. the integral part is then estimated by this line:
`self.i_gap = np.clip(self.i_gap + self.gap_error/self.loop_rate, -10, 10)`
This line clips the cumulative integral error (which is why we're adding the calculated value to the old i error) to -10 and 10. The calculated line itself is based on the gap error and the loop rate, dividing the gap error by the loop rate essentially equals
$$\frac{Gap Error}{loopRate} =  \sum e * \Delta t $$
Where, the gap error is the sum of error, and 1/loopRate equals the Delta Time.
6. The 3 errors is now subtracted from the **Opponent's** Velocity to calculate the speed command. So if the gap is too large, the command would be to  ***Speed Up***, while if the gap is too small, it would have been to ***Slow Down***

The Trailing controller is basically a mini PID controller , the P value is directly proportional to the gap error, the D is the derivative of the error (Which is Velocity) and the I is the cumulative error.

The final command is as follows

$$\text{self.trailing}_{\text{command}} = \text{self.opponent}[2] - (\text{p}_{\text{value}} + \text{i}_{\text{value}} + \text{d}_{\text{value}})$$

which is the expected PID response.