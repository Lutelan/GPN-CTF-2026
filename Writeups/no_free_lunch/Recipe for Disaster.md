##### Challenge Description
_Are you hungry? If so, I have this awesome food ordering app for you. I only ask you not to break it._

__Category__: Pwn(Binary Exploitation)
__Challenge Points__: 
### Solution Process
The challenge files provide us with a 64 bit binary and its source code, `challenge` and `challenge.c`. On running the binary it opens a item ordering prompt which allows us to order upto ten items and finally calculates the total price of the items, it also allows us to send a note to the chef. To see how the program is functioning lets analyze the source code.

On looking at the source code, one immediately sees the interesting function `print_coupon()`, which seems to printing out the flag required for the challenge. Looking for calls to this function in the code we notice that it is called `verify_total()` function, however not always. 
```
void verify_total(int total) {
  if (total < 0) {
    puts("\n[SYSTEM] Pricing error detected! We sincerely apologise for");
    puts("[SYSTEM] the inconvenience. Please accept this coupon:\n");
    print_coupon();
    exit(0);
  }

  printf("\nPlease proceed to the counter to pay $%d. Enjoy your meal!\n",
         total);
}
```
As one can see the function calls `print_coupon()`, if and only if the total price of the items we have bought somehow turns out to be negative, thus one must look at how the total price is calculated to exploit it. 

This leads to the `calculate_total()` function, which looks like this
```
int calculate_total(Item *order, int n_items) {
  int total = 0;
  puts("\n========== YOUR RECEIPT ==========");
  for (int i = 0; i < n_items; i++) {
    printf("  %-26s $%d\n", order[i].item, order[i].price);
    if (order[i].note[0] != '\0')
      printf("    (note: %.31s)\n", order[i].note);
    total += order[i].price;
  }
  puts("----------------------------------");
  printf("  TOTAL                      $%d\n", total);
  puts("==================================");
  return total;
}
```
Here `order` is a pointer to a array of `Item` structs which is defined at the start of the code, each `Item` struct has a `price` attached to it which is a 4 byte integer. If somehow we can overwrite this with something else..

This is where the note part comes in, the struct `Item` is defined as follows
```
typedef struct {
  char item[32];
  char note[32];
  int price;
} Item;
```
As one can see the `note` array lies above the `price` variable, and in the main function the value of `note` is assigned using the unsafe function `gets()`. Thus one can overflow from `note` to `price`.
To make the price negative we rely on that fact that the prices are added, if we can make the price of each item enormous  adding them all up might cross the integer overflow limit and solve the challenge, thus i attempted this by giving the program 36 `Z` characters each time, choosing any items, this results in the following output giving the flag(redacted).
![[Selection_1157.png]]