# Part 9: Custom Swipeable Calendar Component

> Example effect:

![Calendar Selection](img.gif)

In HarmonyOS app development, a calendar component is fundamental for many scenarios such as check-in, booking, and schedule management. Although ArkTS does not provide a built-in full-featured calendar component, you can use basic components like `Swiper` and `ForEach` to create a high-performance, horizontally swipeable custom calendar. This article details the implementation ideas, core code, and best practices. All APIs and terminology refer to the [Huawei Developer Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-ui-components-V5).

## 1. Implementation Ideas and Key Points

1. Use the `Swiper` component to load 3 months of calendars (previous, current, next) managed by the `showDates` array.
2. Initially display the middle page (i.e., index=1), and set `.loop(true)` for infinite swiping.
3. After swiping (`onChange`), only update the data for the left and right months to avoid full UI refresh and improve performance.
4. Both the calendar header and content are rendered dynamically with `ForEach`, supporting custom styles and click events.

## 2. Core Code Example

```ts
/*
 * @Desc: 
 * @Author: qincji
 * @Date: 2024/11/18
 */
import { global } from 'common_utils/src/main/ets/core/Global';
import { dateHelper } from './DateHelper';
import { curves } from '@kit.ArkUI';

@ComponentV2
export struct CalendarView {
  @Event onSelected?: (newDate: Date, oldDate?: Date) => void;
  @Local showDates: Date[] = [dateHelper.lastMoth(), new Date(), dateHelper.nextMoth()];
  @Local selectData: Date = new Date();
  @BuilderParam dayView?: (show: Date) => void;
  private showIndex: number = 1;

  aboutToAppear(): void {
    this.onSelected?.(this.selectData);
  }

  build() {
    Column({ space: 10 }) {
      RelativeContainer() {
        Image($r("app.media.ic_back"))
          .height(40)
          .padding(10)
          .aspectRatio(1)
          .id('last')
          .alignRules({ left: { 'anchor': '__container__', 'align': HorizontalAlign.Start } })
          .onClick(() => {
            this.changeCalender(dateHelper.lastMoth(this.selectData));
          });

        Row() {
          Text(`${this.selectData.getFullYear()}年${this.selectData.getMonth() + 1}月`)
            .fontSize(18)
            .textAlign(TextAlign.Center)
          Image($r("app.media.ic_down"))
            .width(12)
            .aspectRatio(1)
        }
        .id('select')
        .alignRules({
          middle: { 'anchor': '__container__', 'align': HorizontalAlign.Center },
          center: { 'anchor': '__container__', 'align': VerticalAlign.Center }
        })
        .onClick(() => {
          this.getUIContext().showDatePickerDialog({
            selected: this.selectData,
            onDateAccept: (value: Date) => {
              this.changeCalender(value);
            },
          })
        });

        if (!this.isCurMonth()) {
          Text('now')
            .fontSize(12)
            .fontColor(Color.White)
            .backgroundColor($r('app.color.theme_2'))
            .borderRadius(2)
            .padding({ left: 5, right: 5 })
            .margin({ left: 5 })
            .id('now')
            .alignRules({
              left: { 'anchor': 'select', 'align': HorizontalAlign.End },
              center: { 'anchor': '__container__', 'align': VerticalAlign.Center }
            })
            .onClick(() => {
              this.changeCalender(new Date());
            });
        }


        Image($r("app.media.ic_arrow"))
          .width(40)
          .aspectRatio(1)
          .padding(10)
          .id('next')
          .alignRules({ right: { 'anchor': '__container__', 'align': HorizontalAlign.End } })
          .onClick(() => {
            this.changeCalender(dateHelper.nextMoth(this.selectData));
          });
      }
      .height(40)
      .width('100%')

      
      Row() {
        ForEach(["日", "一", "二", "三", "四", "五", "六"], (day: string) => {
          Text(day)
            .width('14%')
            .fontColor('#333333')
            .fontSize(10)
            .textAlign(TextAlign.Center);
        });
      }
      .margin({ bottom: 12 })
      .justifyContent(FlexAlign.SpaceBetween)
      
      Swiper() {
        ForEach(this.showDates, (item: Date) => {
          this.monthView(item)
        })
      }
      .width('100%')
      .height('auto')
      .cachedCount(3)
      .index(1)
      .autoPlay(false)
      .loop(true)
      .itemSpace(0)
      .vertical(false)
      .displayArrow(false, false)
      .indicator(false) 
      .onChange((index: number) => {
        let cur = this.showDates[index];
        this.showIndex = index;
        this.changeCalender(cur);
      })
    }
    .padding({ left: '1%', right: '1%' }) 
  }

  private changeCalender(newDate: Date) {
    this.showDates[this.showIndex] = newDate;
    if (this.showIndex === 0) {
      this.showDates[1] = dateHelper.nextMoth(newDate);
      this.showDates[2] = dateHelper.lastMoth(newDate);
    } else if (this.showIndex === 1) {
      this.showDates[0] = dateHelper.lastMoth(newDate);
      this.showDates[2] = dateHelper.nextMoth(newDate);
    } else { //this.showIndex === 2
      this.showDates[1] = dateHelper.lastMoth(newDate);
      this.showDates[0] = dateHelper.nextMoth(newDate);
    }
    this.onSelected?.(newDate, this.selectData);
    this.selectData = newDate;
  }

  @Builder
  monthView(date: Date) {
    Column({ space: 5 }) {
      ForEach(dateHelper.buildCalendar(date), (week: number[]) => {
        Row() {
          ForEach(week, (day: number) => {
            Stack() {
              if (typeof day === 'number') {
                if (this.dayView === undefined) {
                  Column() {
                    Text(`${day}`)
                      .fontColor('#333333')
                      .fontSize(13)
                      .textAlign(TextAlign.Center)
                      .margin({ top: 5 })
                  }
                  .width('100%')
                  .height('100%')
                  .backgroundColor(this.selectDayColor(date, day))
                  .borderRadius(3)
                } else {
                  this.dayView(new Date(date.getFullYear(), date.getMonth(), day));
                }
              }
            }
            .width('14%')
            .height('100%')
            .padding({left: '1%', right: '1%'})
            .scale(this.getScale(date, day))
            .animation({
              duration: 500,
              curve: Curve.EaseOut,
              iterations: 1,
              playMode: PlayMode.Normal
            })
            .onClick(global.blockQuickClick(() => {
              if (typeof day === 'number') {
                const newDate = new Date(date.getFullYear(), date.getMonth(), day);
                this.onSelected?.(newDate, this.selectData);
                this.selectData = newDate;
                // this.startAnimal();
              }
            }))
          })
        }
        .height(50)
        .justifyContent(FlexAlign.SpaceBetween)
      });
    }
    .transition(TransitionEffect.IDENTITY.combine(TransitionEffect.scale({ x: 2, y: 2 })
      .combine(TransitionEffect.move(TransitionEdge.TOP))
      .animation({ curve: curves.springMotion() })))
  }

  getScale(date: Date, day: number): ScaleOptions {
    if (typeof day === 'number'){
      date = new Date(date.getFullYear(), date.getMonth(), day)
      if (dateHelper.isOneDay(date, this.selectData)) {
        return { x: 1.1, y: 1.1 };
      }
    }

    return { x: 1, y: 1 };
  }

  startAnimal() {
    animateTo({
      duration: 10, onFinish: () => {
      }
    }, () => {
    })
  }

  isCurMonth(): boolean {
    const cur = new Date();
    if (cur.getFullYear() !== this.selectData.getFullYear()) {
      return false;
    }
    if (cur.getMonth() !== this.selectData.getMonth()) {
      return false;
    }
    return true;
  }

  selectDayColor(date: Date, day: number) {
    return dateHelper.isOneDay(this.selectData, new Date(date.getFullYear(), date.getMonth(), day))
      ? '#ff8cdaf1' : Color.Transparent
  }
}
```

## 3. Complete Component Structure

- Year/month switching, jump to current month, left/right arrows, date selection, calendar header, and calendar content are all customizable.
- The `dateHelper` utility generates a two-dimensional array for the calendar, supporting placeholders and same-day checks.

## 4. Best Practices and Official Documentation

- Recommended reading: [ArkTS UI Components Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-ui-components-V5) and [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5).
- It is recommended to encapsulate the calendar as an independent component for reuse and maintenance.
- When swiping, only update necessary data to avoid full refresh.
- Code should include detailed comments, and key exceptions should be logged for troubleshooting.

----

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 