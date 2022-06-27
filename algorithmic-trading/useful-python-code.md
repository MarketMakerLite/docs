---
description: Useful Python code for algo-trading.
---

# Useful Python Code

## General Scripts

#### Convert timestamp to local datetime

```python
from datetime import datetime
from dateutil import tz
def unix_convert(ts):
    ts = int(ts/1000)
    tdate = datetime.utcfromtimestamp(ts).replace(tzinfo=tz.UTC).astimezone(tz.gettz('America/New_York'))
    return tdate
```

or:

```python
import calendar
import time
calendar.timegm(time.strptime('2000-01-01 12:34:00', '%Y-%m-%d %H:%M:%S'))
```

#### Create List of Trading Holidays

```python
from pandas.tseries.holiday import (AbstractHolidayCalendar, Holiday,
                                    USMartinLutherKingJr, USPresidentsDay,
                                    GoodFriday, USMemorialDay,
                                    USLaborDay, USThanksgivingDay,
                                    nearest_workday)

class USTradingCalendar(AbstractHolidayCalendar):
    rules = [
        Holiday('NewYearsDay', month=1, day=1, observance=nearest_workday),
        USMartinLutherKingJr,
        USPresidentsDay,
        GoodFriday,
        USMemorialDay,
        Holiday('USIndependenceDay', month=7, day=4, observance=nearest_workday),
        USLaborDay,
        USThanksgivingDay,
        Holiday('Christmas', month=12, day=25, observance=nearest_workday)
    ]
```

#### Getting a list of tickers from CATNMS

```python
import pandas as pd
df = pd.read_csv("https://files.catnmsplan.com/symbol-master/FINRACATReportableEquitySecurities_EOD.txt", sep="|", header=None, names=["Symbol", "Issue_Name", "Primary_Listing_Mkt", "test_issue_flag"])
df.drop(0, axis=0, inplace=True)
df = df[df.test_issue_flag == 'N'].reset_index(drop=True)
df = df[df.Primary_Listing_Mkt != 'U'].reset_index(drop=True)
symbols = df['Symbol'].to_list()
symbols = [x.replace(' ', '.') for x in symbols]
```

#### Get Last Friday's Date

```python
import datetime
from dateutil.relativedelta import relativedelta, FR

last_friday = datetime.datetime.now() + relativedelta(weekday=FR(-1))
```

## Saving Options Data to Database

{% embed url="https://github.com/MarketMakerLite/TDA/tree/main/options-data" %}
