# Android Firefox Cursor (Bug?)

This code demonstrates a potential cursor behavior bug that only manifests on **Android Firefox** (not in Android Chrome, nor in desktop versions of Firefox).

**Versions**: Android Firefox 56.0 and 57.0b5

## To run

```bash
npm install
npm start # this runs 'serve .'
# access it at http://localhost:5000
```

Or just open `index.html` in your browser. Type numbers in the input field to see the behavior.

## Summary

Upon firing of an `keypress` event, the reported cursor position is wrong in a particular situation.

## More info

We have a telephone input field with a handler function for the `keypress` event. Upon firing of the `keypress` event, we insert a character sequence at the cursor position (given by `event.target.selectionStart`):

* If the user presses 4, we insert a character sequence without '4': `*****`
* If the user presses 6, we insert a character sequence with '6' in the middle: `...6#######`
* For any key other than 4 or 6, we simply insert the the key value  (`event.key`).

After the insertion (which sets `event.target.value` to a new string), the cursor position (`event.target.selectionStart`) is always at the end of the string.

### Bug

After a `keypress` of 6 (for which we inserted `...6#######`), the **next keypress event** reports a cursor position **different than the final cursor position of the previous event**. Namely, the cursor is reported to have moved immediately after the '6' in the string.

This does not occur when the previous `keypress` was 4, for which we inserted a string **not containing '4'** (`*****`). 

### Expected

After setting `event.target.value` to any string and thereby causing the cursor to go to the end of the string, the next `keypress` event should report the same cursor position. (This is the behavior in Android Chrome and desktop Firefox.)

## Screen videos

### Expected

**OS**: macOS Sierra 10.12.6
**Browser**: Firefox Developer Edition, 57.0b5 (64-bit)

Notice that '7', which is typed after '6', gets inserted **at the end** of the string as expected:

![desktop-firefox](./desktop-firefox.gif)

### Buggy

**OS**: Android 7.1.2 (Nexus 5X)
**Browser**: Firefox beta, 57.0b5

Notice that '7', which is typed after '6', gets inserted **after the '6'** in the character sequence, instead of at the end:

![android-firefox](./android-firefox.gif)