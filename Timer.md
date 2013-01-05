  //    *                                   *                                   *
  // F1 *---+---+                           *---+---+                           *
  //    * 1 | 2 |                           * 1 | 2 |                           *
  // ---*       +---------------------------*       +---------------------------*
  //    *                                   *                                   *
  //    *                                   *                                   *
  //    *                                   *                                   *
  //    *                                   *                                   *
  // F2 *       +---+---+---+---+---+       *       +---+---+---+---+---+       *
  //    *       | 1   2   3   4   5 | 1   2 *       | 1   2   3   4   5 | 1   2 *
  // ---*-------+                   +---+---*-------+                   +---+---*
  //    *                                   *                                   *
  //    *         I8080_STATE_TIME          *         I8080_STATE_TIME          *
  //    *<--------------------------------->*<--------------------------------->* 

``` c
#define I8080_FREQ          2
#define I8080_STATE_TIME    (1 / I8080_FREQ)        // 0.5 (500ns)
#define I8080_TIMER_TICK    (I8080_STATE_TIME / 9)  // ~0.055 (~55ns)

#define PIC32_FREQ          80
#define PIC32_TICK          (1 / PIC32_FREQ)        // 0.0125 (12.5ns)

#define I8080_TIMER_PRESCALE (I8080_TIMER_TICK / PIC32_TICK + 1)

#define I8080_TIMER_STATES   9
static int phi1[I8080_TIMER_STATES] = { 1, 1, 0, 0, 0, 0, 0, 0, 0 };
static int phi2[I8080_TIMER_STATES] = { 0, 0, 1, 1, 1, 1, 1, 0, 0 };
static int i8080_timer_state = 0;

#define I8080_PHI1  PORTx.??
#define I8080_PHI2  PORTx.??

// This routine should be called every PIC32_TICK.
void __ISR(_TIMER_1_VECTOR, ipl1) i8080_timer() {
  if (TMR1 < I8080_TIMER_PRESCALE) return;
  TMR1 = 0;
  I8080_PHI1 = phi1[i8080_timer_state];
  I8080_PHI2 = phi2[i8080_timer_state];
  ++i8080_timer_state;
  if (i8080_timer_state > I8080_TIMER_STATES)
    i8080_timer_state = 0;
}
```
