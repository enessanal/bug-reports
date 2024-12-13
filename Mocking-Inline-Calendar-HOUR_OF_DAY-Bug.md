
# Bug Report: Static Mocking of `Calendar.getInstance()` Returns Incorrect `HOUR_OF_DAY`

## Description
When using `MockedStatic` to mock `Calendar.getInstance()`, the `HOUR_OF_DAY` field of the mocked `Calendar` instance does not return the expected value. Other fields like `DAY_OF_MONTH` and `MINUTE` are correctly mocked, but `HOUR_OF_DAY` appears to return `1` (current hour) regardless of the mocked value.

---

## Steps to Reproduce
Run the following test case with `mockito-inline`:

```java
@Test
void testMockedStaticCalendar_Isolated() {
    Calendar mockedToday = Calendar.getInstance();
    mockedToday.set(Calendar.YEAR, 2024);
    mockedToday.set(Calendar.MONTH, Calendar.DECEMBER);
    mockedToday.set(Calendar.DAY_OF_MONTH, 9);
    mockedToday.set(Calendar.HOUR_OF_DAY, 17); // select an hour except current hour
    mockedToday.set(Calendar.MINUTE, 15);

    try (MockedStatic<Calendar> mockedCalendar = mockStatic(Calendar.class)) {
        mockedCalendar.when(Calendar::getInstance).thenReturn(mockedToday);

        Calendar now = Calendar.getInstance();
        assertEquals(9, now.get(Calendar.DAY_OF_MONTH));        // Passes
        assertEquals(17, now.get(Calendar.HOUR_OF_DAY));       // Fails
        assertEquals(15, now.get(Calendar.MINUTE));            // Passes
    }
}
```

---

## Expected Behavior
- `Calendar.getInstance()` should return the mocked `Calendar` instance.
- The `HOUR_OF_DAY` field of the mocked instance should return the value set during mocking (e.g., `17`).

---

## Actual Behavior
- `DAY_OF_MONTH` and `MINUTE` fields return the correct mocked values.
- `HOUR_OF_DAY` always returns current hour: `1`, regardless of the mocked value.

---

## Environment
- Mockito version: `mockito-inline 5.x` (or 4.x for Java 8)
- Java version: `OpenJDK 11` (or `Java 8`)
- Test framework: `JUnit 5`

---

## Additional Information
The issue was also reproduced in a simplified test case:

```java
@Test
void testMockedStaticCalendar() {
    int day = 9;
    int hour = 17; // select an hour except current hour
    int minute = 15;
    Calendar mockedToday = Calendar.getInstance();
    mockedToday.set(Calendar.DAY_OF_MONTH, day);
    mockedToday.set(Calendar.HOUR_OF_DAY, hour);
    mockedToday.set(Calendar.MINUTE, minute);

    try (MockedStatic<Calendar> mockedCalendar = mockStatic(Calendar.class)) {
        mockedCalendar.when(Calendar::getInstance).thenReturn(mockedToday);

        assertEquals(day, Calendar.getInstance().get(Calendar.DAY_OF_MONTH)); // Passes
        assertEquals(hour, Calendar.getInstance().get(Calendar.HOUR_OF_DAY)); // Fails
        assertEquals(minute, Calendar.getInstance().get(Calendar.MINUTE));    // Passes
    }
}
```

---

## Conclusion
This appears to be a bug in `MockedStatic` when mocking `Calendar` objects, specifically affecting the `HOUR_OF_DAY` field. 
