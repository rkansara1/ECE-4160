---
title: "Lab 8: Stunts"
---

In this lab I set out to achieve Task A. To peform Task A the robot needs to drive towards the wall, execute a flip when it is on the black pad, and then drive back past the starting line. To achieve this I modified the code I had written for Lab 7. The only addition was a piece of code that caused the robot to flip when it got close enough and drive backwards. In my testing, I found that to have the robot stop on the black pad it needed to execute the stop at around 900 mm away from the wall.

<script src="https://gist.github.com/rkansara1/5c67fd178163605903b036a018377451.js"></script>


### Trial 1

<iframe width="560" height="315" src="https://www.youtube.com/embed/hbZr8_G3QKs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

![trial1 distance](https://user-images.githubusercontent.com/123790450/232373492-062093e9-8cfa-4219-b6e7-08f6c1f4fc16.png)
![trial 1 motor input](https://user-images.githubusercontent.com/123790450/232373502-e04728e1-16e4-4e27-96e0-31e783b90f8a.png)

Something to note here, as you can see the distance graph stops at 900 mm. This is because in the above code the sensor collection stops once the robot reaches the stopping distance to the wall and then it just executes the flip. If sensor collection was occuring the robot would sense 4000 mm because there are no obstacles in front of it. Additionally the motor input after the stopping distance is the motor inputs used in the commands.

### Trial 2
<iframe width="399" height="710" src="https://www.youtube.com/embed/lrdXg8zvJ9I" title="ECE 4160 Lab 8 Trial 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

![trial 2 distance](https://user-images.githubusercontent.com/123790450/232375123-0b298b9b-7524-44ed-8e66-1b937c3cbb1f.png)
![trial2 motor input vs time](https://user-images.githubusercontent.com/123790450/232375160-6af2f83e-0dd4-415b-8ab9-99edea0bf2cd.png)

### Trial 3
<iframe width="399" height="710" src="https://www.youtube.com/embed/p8zhkRKGQcQ" title="ECE 4160 Lab 8 Trial 3" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

![trial 3 distance](https://user-images.githubusercontent.com/123790450/232375270-3cc66a1d-fca1-492c-b6f9-9f6df20f91e6.png)
![trial 3 motor](https://user-images.githubusercontent.com/123790450/232375273-0e4fa13e-533f-4bf9-876e-86068c74f7c7.png)
