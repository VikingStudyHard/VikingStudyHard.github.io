---

title: 【软件测试】03 图覆盖

date: 2018-03-24

categories: [软件测试, 本科课程]

tags: 

 - SoftwareTest

description: Use the following method printPrimes() for questions a–d. 并基于Junit及Eclemma(jacoco)实现一个主路径覆盖的测试。

---


# Question

```java
/***********************************
 * Finds and prints n prime integers
 * Jeff Offutt, Spring 2003
************************************/
private static void printPrimes (int n){
    int curPrime;        // Value currently considered for primeness
    int numPrimes;        // Number of primes found so far.
    boolean isPrime;    // Is curPrime prime?
    int [] primes = new int [MAXPRIMES]; // The list of prime numbers.
    // Initialize 2 into the list of primes.
    primes[0] = 2;
    numPrimes = 1;
    curPrime = 2;
    while(numPrimes < n){
        curPrime++;        //next number to consider ...
        isPrime = true;
        for(int i = 0; i <= numPrimes-1; i++){
            if(isDivisible(primes[i], curPrime)){
                isPrime = false;
                break;
            }
        }
        if(isPrime){
            primes[numPrimes] = curPrime;
            numPrimes++;
        }
    } // end while
    //Print all the primes out.
    for(int i = 0; i <= numPrimes-1; i ++){
        System.out.println("Prime: "+ primes[i]);
    }
} // end printPrimes
```
a) Draw the control flow graph for the printPrimes() method.

b) Consider test cases t1 = (n = 3) and t2 = (n = 5). Although these tour the prime paths in printPrimes(), they do not necessarily find the same faults. Design a simple fault that t2 would be more likely to discover than t1 would.

c) For printPrimes(), find a test case such that the corresponding test path visits the edge that connects the beginning of the while statement to the for statement without going through the body of the while loop.

d) Enumerate the test requirements for node coverage, edge coverage, and prime path coverage for the graph for printPrimes().
e) 基于Junit及Eclemma(jacoco)实现一个主路径覆盖的测试。
# Answer
##a)control flow graph
![flow.png](/images/STH3/flow.png)
##b)
If constant variable MAXPRIMES equals 4,it will occur a fault when n equals 5 but will not if n equals 3.
##c)without going through the body of the while loop
n=1
##d)
###node coverage
TR={1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}
Test Path:[1,2,3,4,5,6,7,5,6,8,9,10,2,11,12,13,14,12,15]
###edge coverage
TR={(1,2),(2,3),(2,11),(3,4),(4,5),(5,6),(5,9),(6,7),(6,8),(7,5),(8,9),(9,2),(9,10),(10,2),(11,12),(12,13),(12,15),(13,14),(14,12)}
Test Path:[1,2,3,4,5,6,7,5,9,2,11,12,13,14,12,15] [1,2,3,4,5,6,8,9,10,2,11,12,15]
###prime path coverage

**Simple paths:**
![simplePath.png](/images/STH3/simplePath.png)


**Prime paths**
{[5,6,7,5]
[6,7,5,6]
[7,5,6,7] 
[12,13,14,12]
[13,14,12,13]
[1,2,11,12,15] 
[1,2,3,4,5,6,7]
[2,3,4,5,9,2]
[3,4,5,9,2,3] 
[4,5,9,2,3,4]
[5,9,2,3,4,5]
[9,2,3,4,5,9] 
[14,12,13,14]
[13,14,12,15]
[2,3,4,5,9,10,2]
[3,4,5,9,10,2,3]
[4,5,9,10,2,3,4]
[5,9,10,2,3,4,5]
[9,10,2,3,4,5,9]
[10,2,3,4,5,9,10]
[2,3,4,5,6,8,9,2]
[3,4,5,6,8,9,2,3]
[4,5,6,8,9,2,3,4] 
[5,6,8,9,2,3,4,5]
[6,8,9,2,3,4,5,6] 
[8,9,2,3,4,5,6,8]
[9,2,3,4,5,6,8,9]
[1,2,3,4,5,6,8,9,10]
[2,3,4,5,6,8,9,10,2]
[3,4,5,6,8,9,10,2,3]
[4,5,6,8,9,10,2,3,4] 
[5,6,8,9,10,2,3,4,5]
[6,8,9,10,2,3,4,5,6]
[8,9,10,2,3,4,5,6,8]
[9,10,2,3,4,5,6,8,9]
[7,5,6,8,9,2,11,12,15] 
[6,7,5,9,2,11,12,15] 
[6,7,5,9,10,2,11,12,13,14]
[10,2,3,4,5,6,8,9,10]
[3,4,5,9,2,11,12,13,14]
[3,4,5,6,8,9,2,11,12,15]
[3,4,5,6,8,9,10,2,11,12,15]
[3,4,5,6,8,9,2,11,12,13,14]
[3,4,5,6,8,9,10,2,11,12,13,14]}
##e)
Main.java
```java
public class Main {
    private static final int MAXPRIMES = 5;

    private static boolean isDivisible(int a, int b) {
        if (b % a == 0)
            return true;
        return false;
    }

    public static int[] printPrimes(int n) {
        int curPrime;        // Value currently considered for primeness
        int numPrimes;        // Number of primes found so far.
        boolean isPrime;    // Is curPrime prime?
        int[] primes = new int[MAXPRIMES]; // The list of prime numbers.
        // Initialize 2 into the list of primes.
        primes[0] = 2;
        numPrimes = 1;
        curPrime = 2;
        while (numPrimes < n) {
            curPrime++;        //next number to consider ...
            isPrime = true;
            for (int i = 0; i <= numPrimes - 1; i++) {
                if (isDivisible(primes[i], curPrime)) {
                    isPrime = false;
                    break;
                }
            }
            if (isPrime) {
                primes[numPrimes] = curPrime;
                numPrimes++;
            }
        } // end while
        //Print all the primes out.
        for (int i = 0; i <= numPrimes - 1; i++) {
            System.out.println("Prime: " + primes[i]);
        }
        return primes;
    } // end printPrimes
}
```
MainTest.java
```java
import static org.junit.Assert.*;
import org.junit.Test;

public class MainTest {
    @Test
    public void printPrimes() throws Exception {
        int [] a = new int [] {2, 3, 5, 7, 11};
        assertArrayEquals(a, Main.printPrimes(5));
    }

}
```

设置测试引擎：JaCoCo
![Jacoco.png](/images/STH3/Jacoco.png)

结果：
![result.png](/images/STH3/result.png)


