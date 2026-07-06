This is my implementation + breakdown + documentation of KD Tree using VEX
[See my code files]()

This is a recursive algorithm, which means it uses memory overhead (recursion stack) to store the result of previous function calls. 

## A Note On Recursion in VEX

On searching online, people have come up with lots of great methods to make recursion work in VEX. From mimicing a stack using an array, to using multiple pointers. The method that is most intuitive to me is using groups. I learnt this from [Junichiro Horikawa's INCREDIBLE playlist](https://www.youtube.com/watch?v=V5i6KM_-8X0) - lesson 26 talks about using groups for recursion with lots of useful examples. Honestly, I went from zero to being able to write and execute full projects in Houdini on my own because of religiously following this one guy's tutorials.
So now let's see how I got the KD Tree algorithm to work using this trick with the group feature in Houdini!

BTW, you can find the KD tree algorithm itself nicely explained here: [https://www.baeldung.com/cs/k-d-trees](https://www.baeldung.com/cs/k-d-trees)

## The Node Setup

## Coding the algorithm + Breakdown of Code


Sources ♥️
1. [https://www.baeldung.com/cs/k-d-trees](https://www.baeldung.com/cs/k-d-trees)
2. [https://sergeneren.com/2018/09/23/recursion-in-vex/](https://sergeneren.com/2018/09/23/recursion-in-vex/)
3. [https://www.youtube.com/watch?v=V5i6KM_-8X0](https://www.youtube.com/watch?v=V5i6KM_-8X0)
