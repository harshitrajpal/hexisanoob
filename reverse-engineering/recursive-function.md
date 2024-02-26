# Recursive Function

binary: recurse.bin (elf64)

<figure><img src="../.gitbook/assets/image (395).png" alt=""><figcaption></figcaption></figure>

Simplified code in C looks like:

```
int main(void)

{
  undefined8 var1;
  int a;
  int b;
  
  setup();
  printf("Enter your initial values: ");
  scanf("%d %d",&a,&b);
  if ((a == 0) || (b == 0)) {
    puts("Values must both be non-zero!");
    var1 = 1;
  }
  else {
    var1 = recurse(a,b,0,1);
    if ((int)var1 == 0) {
      puts("Nope!");
    }
    else {
      puts("Correct!");
      print_flag();
    }
    var1 = 0;
  }
  return var1;
}


```

1. The application takes 2 numbers as input
2. If the evaluation of recurse() is non-zero, the flag is printed

Inspecting the recurse function:

<figure><img src="../.gitbook/assets/image (396).png" alt=""><figcaption></figcaption></figure>

Simplified version:

```
undefined8 recurse(int a,int b)

{
  int iVar1;
  undefined8 uVar2;
  
  iVar1 = ((b + a) - 0) - 1;
  if ((0 == c_goal) && (b + a == sum_goal)) {
    uVar2 = 1;
  }
  else if ((0 < c_goal) && (((a < iVar1 || (b < iVar1)) || (0 < iVar1)))) {
    uVar2 = absolutely_not_useless_fn(b,b + a,0 + 1,iVar1);
  }
  else {
    uVar2 = 0;
  }
  return uVar2;
}

```

Now what is c\_goal and sum\_goal? Based on that I'll revise this code again

<figure><img src="../.gitbook/assets/image (397).png" alt=""><figcaption></figcaption></figure>

And then there is one more function used in the recurse code:

<figure><img src="../.gitbook/assets/image (398).png" alt=""><figcaption></figcaption></figure>

So, a lot of math is involved. I applied all these functions and replaced values and figured out some ranges of data that could fit the answer. Basically,

```
#include<stdlib.h>
#include<stdio.h>

int c_goal=16;
int sum_goal=116369;
int useful_variable=1020;
int another_useful_one=2010;

int absolutely_not_useless_fn(int, int, int, int);

int recurse(int a, int b, int c, int d){
        int i;
        int j;

        i = ((b+a)-c) - d;
        if((c == c_goal)&&(b+a==sum_goal)){
                j=1;
        }
        else if((c<c_goal)&&(((a<i || (b<i)) || (c<i)))){
                j = absolutely_not_useless_fn(b,b+a,c+1,i);
        }
        else{
                j=0;
        }
        return j;
}

int absolutely_not_useless_fn(int l, int m, int n, int o){
        useful_variable = (o+l) - n;
        another_useful_one = m+l;
        recurse(m,another_useful_one,n+1,((another_useful_one+useful_variable)/3)*2);
        return 1;
}

int main()
{
        int x;
        int num1=1, num2=1;
        //printf("Enter your initial values: ");
        //scanf("%d %d",&num1,&num2);
        for(num1=1;num1<=50;num1++){
                for(num2=1;num2<=50;num2++){
                        x=recurse(num1,num2,0,1);
                        if((int)x==0){
                                puts("Nope!");
                        }
                        else{
                                puts("Correct!");
                        }
                }
        }
        return 0;

}
```



1. Range of input is between 1 and 50.
2. Enough, I can't do so much math for this.
3. I'll brute force :)

Takes input like this:

<figure><img src="../.gitbook/assets/image (399).png" alt=""><figcaption></figcaption></figure>

I can create a BASH one-liner to bruteforce this.

for i in {1..50}; do for j in {1..50}; do echo $i $j; echo $i $j | ./recurse.bin; done; done > output

<figure><img src="../.gitbook/assets/image (400).png" alt=""><figcaption></figcaption></figure>

So, correct values are 13 and 37
