<template>
    <div>
        <CalendarHeader @closePopup="showPopup = false"></CalendarHeader>

        <div class="calendar">
            <div v-for="cell of cells"
                 :key="cell.id"
                 class="cell"
                 @click="showCalendarPopup($event, {event: null, cell})"
                 :style="{ backgroundImage: thumbnails[currentMonth * 31 + cell.id - 1] }">
                <strong>{{ cell.id }}</strong>

                <div class="events">
                    <span v-for="event of cell.events"
                          :key="event.id"
                          @click.stop="showCalendarPopup($event, {event, cell})"
                          class="event">
                        {{ event.label }}
                    </span>
                </div>
            </div>

            <div class="footer">
                <div class="infos"
                     v-if="fetchingHeroes">
                    <Loader></Loader>
                    <span>Le super signal a été envoyé, les héros arrivent...</span>
                </div>

                <a href="http://marvel.com">Data provided by Marvel. © {{ new Date().getUTCFullYear() }} MARVEL</a>
            </div>
        </div>

        <transition name="bounce">
            <CellDetailPopup v-if="showPopup"
                             :clickedCell="clickedCell"
                             :readableDate="readableDate"
                             :selectedEvent="selectedEvent"
                             @closePopup="showPopup = false">
            </CellDetailPopup>
        </transition>
    </div>
</template>

<script lang="ts">
  import { Component, Prop, Vue } from 'vue-property-decorator';
  import { CalendarEvent } from '@/models/CalendarEvent';
  import Loader from '@/components/Loader.vue';
  import CellDetailPopup from '@/modules/calendar/components/CellDetailPopup.vue';
  import CalendarHeader from '@/modules/calendar/components/CalendarHeader.vue';
  import { State, Action, Getter, namespace } from 'vuex-class';
  import { Cell, ICell } from '@/models/Cell';

  const moduleNamespace = '$_calendar';
  const storeModule = namespace(moduleNamespace);

  @Component({
    name: 'Calendar',
    components: {
      CellDetailPopup,
      Loader,
      CalendarHeader,
    }
  })
  export default class ArticlesList extends Vue {
    @storeModule.Getter events!: CalendarEvent[];
    @storeModule.Getter heroes!: any[];
    @storeModule.Getter fetchingHeroes!: boolean;
    @storeModule.Getter currentDay!: number;
    @storeModule.Getter currentMonth!: number;
    @storeModule.Getter currentYear!: number;

    showPopup = false;
    clickedCell: ICell = new Cell();
    readableDate!: string;
    selectedEvent!: CalendarEvent;

    get cells() {
      const daysInMonth = this.daysInMonth(this.currentMonth, this.currentYear);
      return this.buildDaysArray(daysInMonth);
    }

    get thumbnails() {
      return this.heroes.map(hero => `url(${hero.thumbnail.path}.${hero.thumbnail.extension})`);
    }

    private showCalendarPopup(e: any, payload = { event: new CalendarEvent(), cell: new Cell() }): void {
      const popupAlreadyInitialized = this.showPopup;
      this.showPopup = false;
      const el = e.target.getBoundingClientRect();

      this.clickedCell = {
        id: payload.cell.id,
        x: el.left,
        y: el.top,
        w: el.width,
        h: el.height,
        events: payload.cell.events,
      };

      this.readableDate = this.createReadableDate(payload.cell.id);
      this.selectedEvent = payload.event;

      if (popupAlreadyInitialized) {
        setTimeout(() => this.showPopup = true, 300);
      } else {
        this.showPopup = true
      }
    }

    private buildDaysArray(daysInMonth: number): object[] {
      const daysArray = [];

      for (let i = 1; i <= daysInMonth; i++) {
        const events = this.events.filter(event => {
          const date = new Date(event.date_start);
          return date.getDate() === i && date.getMonth() === this.currentMonth;
        });

        daysArray.push({
          id: i,
          events,
        });
      }

      return daysArray;
    }

    private daysInMonth(iMonth: number, iYear: number): number {
      return 32 - new Date(iYear, iMonth, 32).getDate();
    }

    private createReadableDate(dayInMonth: number): string {
      const date = new Date(this.currentYear, this.currentMonth, dayInMonth);
      const days = ['Dimanche', 'Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi'];
      const months = ['Janvier', 'Février', 'Mars', 'Avril', 'Mai', 'Juin', 'Juillet', 'Août', 'Septembre', 'Octobre', 'Novembre', 'Décembre'];
      const day = days[date.getDay()];
      const month = months[date.getMonth()];

      return `${day} ${dayInMonth} ${month} ${this.currentYear}`;
    }
  }
</script>

<style scoped lang="scss">
    .calendar {
        height: 90vh;
        display: grid;
        grid-template-columns: repeat(7, 1fr);
        grid-template-rows: repeat(5, 1fr);
        grid-column-gap: 2px;
        grid-row-gap: 2px;
        padding: 3px;
        box-sizing: border-box;
        cursor: pointer;
        overflow-y: hidden;

        .cell {
            background-size: cover;
            padding: 5px;
            box-sizing: border-box;
            border: solid black;
            transition: all .2s ease-in;
            display: flex;
            flex-direction: column;
            overflow-y: auto;
            align-items: flex-end;

            &:nth-child(n) {
                border-width: 3px 3px 5px 5px;
                border-radius:4% 95% 6% 95%/95% 4% 92% 5%;
                transform: rotate(-2deg);
            }

            &:nth-child(2n) {
                border-width: 3px 4px 3px 5px;
                border-radius:95% 4% 92% 5%/4% 95% 6% 95%;
                transform: rotate(2deg);
            }

            &:nth-child(3n) {
                border-width: 5px 3px 3px 5px;
                border-radius:95% 4% 97% 5%/4% 94% 3% 95%;
                transform: rotate(2deg);
            }

            &:hover {
                @include shadow();
                transform: scale(1.1);
                z-index: 1;
            }

            strong {
                color: white;
                text-shadow: -1px 1px 0 black;
                font-size: 1.3rem;
                font-weight: bold;
                align-self: flex-start;
            }

            .events {
                .event {
                    background: white;
                    border: 3px solid;
                    font-weight: bold;
                    padding: 3px;
                    border-radius: 3px;

                    &:hover {
                        background: $yellow;
                    }
                }
            }
        }

        .footer {
            grid-column-start: 4;
            grid-column-end: 8;
            display: flex;
            flex-direction: column;
            justify-content: flex-end;

            .infos {
                display: flex;
                align-self: center;
                align-items: center;

                span {
                    font-weight: bold;
                    font-size: 1.4rem;
                    margin-left: 1rem;
                }
            }

            a {
                align-self: flex-end;
                margin-top: 1rem;
            }
        }
    }

    .bounce-enter-active {
        animation: bounce-in .3s;
    }
    .bounce-leave-active {
        animation: bounce-in .3s reverse;
    }
    @keyframes bounce-in {
        0% {
            transform: rotate(2deg) scale(0);
        }
        50% {
            transform: rotate(2deg) scale(1.2);
        }
        100% {
            transform: rotate(2deg) scale(1);
        }
    }
</style>