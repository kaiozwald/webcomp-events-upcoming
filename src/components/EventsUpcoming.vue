<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div
    class="events-widget"
    :class="'theme-' + (options.theme || 'default')"
    :style="{
      'font-family': options.fontName + ', sans-serif',
      'background-color': options.backgroundColor,
    }"
  >
    <header class="events-header">
      <h1 class="events-title">EVENTS</h1>
      <div class="events-datetime">
        <span class="events-date">{{ currentDate() }}</span>
        <span class="events-time">{{ timestamp }}</span>
      </div>
    </header>

    <div class="events-container">
      <!-- Skeleton Loader -->
      <div v-if="isLoading" class="events-list">
        <div class="event-card skeleton-card" v-for="i in 3" :key="'skel-' + i">
          <div class="skeleton-main">
            <div class="skeleton-title"></div>
            <div class="skeleton-subtitle"></div>
          </div>
          <div class="skeleton-meta">
            <div class="skeleton-pill"></div>
            <div class="skeleton-date"></div>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div v-else-if="events.length === 0" class="empty-state">
        <div class="empty-icon-wrapper">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="empty-icon"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="1"
              d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
            />
          </svg>
        </div>
        <h3>No Upcoming Events</h3>
        <p>Check back later for new scheduled events.</p>
      </div>

      <!-- Event List -->
      <transition-group
        v-else
        name="list-stagger"
        tag="div"
        class="events-list"
      >
        <div
          class="event-card list-stagger-item"
          v-for="(event, index) in events"
          :key="'evt-' + index"
          :style="{ animationDelay: index * 75 + 'ms' }"
          @click="toggleExpand(index)"
          @keydown.enter.space.prevent="toggleExpand(index)"
          tabindex="0"
          role="button"
          :aria-expanded="isExpanded(index).toString()"
          :class="{ 'is-expanded': isExpanded(index) }"
        >
          <div class="event-card-main">
            <div class="event-info">
              <h2 class="event-name" v-if="event.webAddress">
                <a :href="event.webAddress" target="_blank" @click.stop>{{
                  event.shortName
                }}</a>
              </h2>
              <h2 class="event-name" v-else>{{ event.shortName }}</h2>
              <div class="event-period">{{ event.dateperiod }}</div>
            </div>

            <div class="event-meta">
              <div
                class="event-location"
                :style="{
                  backgroundImage:
                    'linear-gradient(135deg, ' +
                    options.backgroundColor +
                    ' 0%, var(--theme-dark, #2F5C30) 100%)',
                }"
              >
                <span style="display: flex; align-items: center; gap: 0.4rem">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="meta-icon"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"
                    />
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"
                    />
                  </svg>
                  {{ event.eventLocation }}
                </span>
              </div>
              <div class="event-upcoming">
                <div class="event-upcoming-date">
                  {{ formatDate(event.nextBeginDate) }}
                </div>
                <div class="event-upcoming-time">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="meta-icon"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                    />
                  </svg>
                  {{ event.nextBeginTime }}
                </div>
              </div>
              <div
                class="expand-icon"
                :class="{ 'is-rotated': isExpanded(index) }"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke="currentColor"
                  width="24"
                  height="24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2.5"
                    d="M19 9l-7 7-7-7"
                  />
                </svg>
              </div>
            </div>
          </div>

          <!-- Expanded Details -->
          <transition name="expand">
            <div v-if="isExpanded(index)" class="event-expanded">
              <div class="expanded-header">All Event Dates:</div>
              <div class="expanded-dates">
                <div
                  v-for="(d, dIndex) in event.allDates"
                  :key="dIndex"
                  class="expanded-date-item"
                >
                  <span class="date-day">
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      class="meta-icon"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
                      />
                    </svg>
                    {{ d.From.substring(0, 10) }}
                  </span>
                  <span
                    class="date-time"
                    v-if="d.Begin && d.End && !d.Begin.startsWith('00:00:00')"
                  >
                    {{ d.Begin.substring(0, 5) }} &mdash;
                    {{ d.End.substring(0, 5) }}
                  </span>
                  <span class="date-time" v-else>All day</span>
                  <span class="date-room" v-if="d.roomName">{{
                    d.roomName
                  }}</span>
                </div>
              </div>
            </div>
          </transition>
        </div>
      </transition-group>

      <div class="events-footer">
        <a href="https://opendatahub.com" target="_blank">
          powered by Open Data Hub
          <img
            :src="require('@/assets/icons/NOI_OPENDATAHUB_NEW_WH-01.png')"
            alt="ODH Logo"
          />
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import _ from "lodash";

export default {
  name: "EventsUpcoming",
  props: {
    options: Object,
    default: () => {
      return {};
    },
  },
  data: function () {
    return {
      events: [],
      timestamp: "",
      languages: ["en", "de", "it"],
      currentlanguage: "",
      backgroundcolor: "",
      venueCache: {},
      expandedIndices: [],
      isLoading: true,
    };
  },
  computed: {
    orderedEvents: function () {
      return _.orderBy(this.events, "nextBeginDate");
    },
  },
  created: function () {
    this.currentlanguage = this.options.language;
    this.getNow();
    this.backgroundcolor = this.options.backgroundColor;

    //If no language is set use the rotation
    if (this.options.language == "") {
      this.currentlanguage = "en";
      setInterval(
        this.rotateLanguage,
        this.options.languageRotationInterval * 1000
      );
    }
    this.rotateEvents();
    // create cron job
    setInterval(this.getNow, 1000);
    setInterval(this.rotateEvents, this.options.eventRefreshInterval * 1000);
  },
  methods: {
    async fetchData() {
      this.isLoading = true;
      this.events = [];
      const baseURL = process.env.VUE_APP_TOURISM_BASE_PATH + "/v1/Event?";
      const paramsList = [
        ["begindate", this.formatDateAndTime(new Date())],
        ["locfilter", this.options.locationFilter],
        ["language", this.currentlanguage],
        ["langfilter", this.currentlanguage],
        ["pagesize", this.options.maxEvents ? this.options.maxEvents : 999],
        ["active", true],
        ["sort", this.options.eventSortmode],
        ["origin", "webcomp-events-upcoming"],
      ];

      if (this.options.source && this.options.source !== "null") {
        let sourceValue = Array.isArray(this.options.source)
          ? this.options.source.join(",")
          : this.options.source;
        paramsList.push(["source", sourceValue]);
      }
      if (this.options.tags) {
        paramsList.push(["tagfilter", this.options.tags]);
      }
      if (this.options.publishedOn) {
        paramsList.push(["publishedon", this.options.publishedOn]);
      }

      const params = new URLSearchParams(paramsList);

      try {
        const response = await fetch(baseURL + params, {
          method: "GET",
          headers: {
            "Content-Type": "application/json",
          },
        });
        if (!response.ok)
          throw new Error(`HTTP error! Status: ${response.status}`);

        const json = await response.json();
        const items = json.Items || [];

        for (let i = 0; i < items.length; ++i) {
          let element = items[i];
          let startDate = new Date(element.DateBegin);
          let endDate = new Date(element.DateEnd);
          let nextbegin = this.getNextBeginDate(element.EventDate);

          let eventLocation = "";
          if (
            element.Source === "lts" ||
            !(element.VenueIds && element.VenueIds.length > 0)
          ) {
            eventLocation = this.getLocationToShow(
              element,
              this.options.locationToShow,
              this.currentlanguage
            );
          } else {
            let venueId = element.VenueIds[0];
            let venueRoomId = null;
            if (
              this.options.showRoomOnMain === "true" ||
              this.options.showRoomOnMain === true
            ) {
              venueRoomId = nextbegin[2];
            }

            let venueName = await this.getVenueName(
              venueId,
              venueRoomId,
              this.currentlanguage
            );
            eventLocation =
              venueName ||
              this.getLocationToShow(
                element,
                this.options.locationToShow,
                this.currentlanguage
              );
          }

          let event = {
            shortName:
              element.Detail?.[this.currentlanguage]?.Title ?? "no title",
            eventLocation: eventLocation,
            webAddress: element.ContactInfos?.[this.currentlanguage]?.Url,
            dateperiod: this.getPeriod(
              startDate,
              endDate,
              element.EventAdditionalInfos?.[this.currentlanguage]
            ),
            startDate: this.formatDate(startDate),
            endDate: this.formatDate(endDate),
            nextBeginDate: nextbegin[0],
            nextBeginTime: nextbegin[1],
            allDates: element.EventDate,
            room: "", // Will be populated in Date parsing if available
          };

          if (event.allDates) {
            for (let d of event.allDates) {
              if (d.VenueRoomDetailsIds && d.VenueRoomDetailsIds.length > 0) {
                let rootVenueId =
                  element.VenueIds && element.VenueIds.length > 0
                    ? element.VenueIds[0]
                    : null;
                let roomName = await this.getVenueName(
                  rootVenueId,
                  d.VenueRoomDetailsIds[0],
                  this.currentlanguage
                );
                if (roomName && roomName.includes(" - ")) {
                  d.roomName = roomName.split(" - ")[1];
                } else {
                  d.roomName = roomName;
                }
              }
            }
          }

          this.events.push(event);
        }
      } catch (error) {
        console.error("Error fetching events:", error);
      } finally {
        this.isLoading = false;
      }
    },
    rotateEvents() {
      // first update events
      this.fetchData();
    },
    async getVenueName(venueId, venueRoomId, language) {
      if (!venueId) return null;
      if (!this.venueCache[venueId]) {
        try {
          const response = await fetch(
            process.env.VUE_APP_TOURISM_BASE_PATH + "/v1/Venue/" + venueId
          );
          if (response.ok) {
            this.venueCache[venueId] = await response.json();
          } else {
            this.venueCache[venueId] = null;
          }
        } catch (e) {
          console.error("Error fetching venue", e);
          this.venueCache[venueId] = null;
        }
      }

      const venue = this.venueCache[venueId];
      if (!venue) return null;

      let venueName = venue.Detail?.[language]?.Title || venue.Shortname || "";

      if (venueRoomId && venue.RoomDetails) {
        let room = venue.RoomDetails.find((r) => r.Id === venueRoomId);
        if (room) {
          let roomName = room.Detail?.[language]?.Title || room.Shortname || "";
          if (roomName) {
            return venueName + " - " + roomName;
          }
        }
      }

      return venueName;
    },
    rotateLanguage() {
      let index = this.languages.indexOf(this.currentlanguage) + 1;

      if (index >= this.languages.length) index = 0;
      this.currentlanguage = this.languages[index];

      console.log("language changed to: " + this.currentlanguage);
      // first update events
      this.fetchData();
    },
    currentDate() {
      let locale = "en-GB";
      if (this.currentlanguage == "de") locale = "de-DE";
      if (this.currentlanguage == "it") locale = "it-IT";
      const current = new Date();
      return current
        .toLocaleDateString(locale, {
          month: "long",
          year: "numeric",
          day: "numeric",
        })
        .replace(",", "")
        .toUpperCase();
    },
    getLocationToShow(event, locationToShow, language) {
      if (locationToShow == "district")
        return event.LocationInfo?.DistrictInfo?.Name?.[language];
      if (locationToShow == "municipality")
        return event.LocationInfo?.MunicipalityInfo?.Name?.[language];
      if (locationToShow == "tourismorganization")
        return event.LocationInfo?.TvInfo?.Name?.[language];
      if (locationToShow == "region")
        return event.LocationInfo?.RegionInfo?.Name?.[language];
      if (locationToShow == "location")
        return event.EventAdditionalInfos?.[language]?.Location;
      else return event.LocationInfo?.DistrictInfo?.Name?.[language];
    },
    getNextBeginDate(eventdate) {
      let nextbegindate = null;
      let nextbegintime = null;
      let nextbeginroomid = null;
      let now = Date.now();
      let tempdifference = 9999999999999;

      let allday = { de: "ganztägig", it: "giornata intera", en: "all day" };
      let noinfo = { de: "keine angabe", it: "senza info", en: "no info" };

      eventdate.forEach((value) => {
        var fullstartdate = new Date(
          value.From.replace("00:00:00", value.Begin)
        );
        var fullenddate = new Date(value.To.replace("00:00:00", value.End));

        //If Eventdate is defined as single Days
        if (value.From == value.To) {
          //calculate timediff from now and get closest greater than
          var difference = fullstartdate - now;
          var hasended = fullenddate - now;

          if (hasended >= 0 && difference <= 0) difference = 0;

          //Only if has not ended and the difference is the minimum
          if (hasended >= 0 && difference <= tempdifference) {
            nextbegindate = new Date(value.From);
            nextbeginroomid =
              value.VenueRoomDetailsIds && value.VenueRoomDetailsIds.length > 0
                ? value.VenueRoomDetailsIds[0]
                : null;

            if (
              value.Begin.startsWith("00:00") &&
              value.End.startsWith("23:59")
            )
              nextbegintime = allday[this.currentlanguage];
            else
              nextbegintime =
                value.Begin.substring(0, 5) + " - " + value.End.substring(0, 5);
            tempdifference = difference;
          }
        }
        //If interval is valid set datetime now as date
        else {
          if (new Date(value.From) <= now && new Date(value.To) >= now) {
            nextbegindate = now;
            nextbegintime = noinfo[this.currentlanguage];
          } else if (new Date(value.From) > now) {
            difference = new Date(value.From) - now;
            if (difference <= tempdifference) {
              nextbegindate = new Date(value.From);
              nextbegintime = noinfo[this.currentlanguage];
              nextbeginroomid =
                value.VenueRoomDetailsIds &&
                value.VenueRoomDetailsIds.length > 0
                  ? value.VenueRoomDetailsIds[0]
                  : null;
              tempdifference = difference;
            }
          }
        }
      });

      return [nextbegindate, nextbegintime, nextbeginroomid];
    },
    getPeriod(startDate, endDate, additionalinfo) {
      const start = moment(startDate);
      const end = moment(endDate);

      let period = "";
      if (start.isValid() && end.isValid()) {
        period = start.format("DD-MM-YYYY");
        if (start.format("DD-MM-YYYY") !== end.format("DD-MM-YYYY")) {
          period += " - " + end.format("DD-MM-YYYY");
        } else if (additionalinfo != null && additionalinfo.Location) {
          period = additionalinfo.Location;
        }
      } else if (start.isValid()) {
        period = start.format("DD-MM-YYYY");
      } else if (additionalinfo != null && additionalinfo.Location) {
        period = additionalinfo.Location;
      }

      return period;
    },
    formatTime(date) {
      if (!date) return "";
      const m = moment(date);
      return m.isValid() ? m.format("HH:mm") : "";
    },
    formatDate(date) {
      if (!date) return "";
      const m = moment(date);
      return m.isValid() ? m.format("DD-MM-YYYY") : "";
    },
    formatDateAndTime(date) {
      if (!date) return "";
      const m = moment(date);
      return m.isValid() ? m.format("YYYY-MM-DD HH:mm") : "";
    },
    getNow: function () {
      const today = new Date();
      const time =
        today.getHours() +
        ":" +
        (today.getMinutes() < 10 ? "0" : "") +
        today.getMinutes();
      this.timestamp = time;
    },
    toggleExpand(index) {
      if (this.options.expanded === "true" || this.options.expanded === true)
        return; // Always expanded

      const pos = this.expandedIndices.indexOf(index);
      if (pos !== -1) {
        this.expandedIndices.splice(pos, 1);
      } else {
        this.expandedIndices.push(index);
      }
    },
    isExpanded(index) {
      if (this.options.expanded === "true" || this.options.expanded === true)
        return true;
      return this.expandedIndices.includes(index);
    },
  },
};
</script>

<style scoped>
.events-widget {
  width: 100%;
  min-height: 100vh;
  color: var(--text-main, #2d3748);
}

.events-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 2.5rem 2.5rem 1rem 2.5rem;
  background: transparent;
  color: var(--header-text, #ffffff);
}

.events-title {
  font-size: 3.5rem;
  font-weight: 800;
  letter-spacing: -0.05em;
  color: var(--header-text, #ffffff);
  margin: 0;
}

.events-datetime {
  display: flex;
  align-items: baseline;
  gap: 1rem;
}

.events-date {
  font-size: 2.25rem;
  font-weight: 700;
  color: var(--header-text, #ffffff);
}

.events-time {
  font-size: 1.5rem;
  color: rgba(255, 255, 255, 0.85);
  font-weight: 600;
}

.events-container {
  padding: 1rem 2.5rem 2.5rem 2.5rem;
}

.events-list {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.event-card {
  background: var(--card-bg, rgba(255, 255, 255, 0.85));
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border: 1px solid rgba(255, 255, 255, 0.6);
  border-radius: 16px;
  padding: 1.75rem 2.25rem;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05),
    inset 0 1px 0 rgba(255, 255, 255, 0.8);
  transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275),
    box-shadow 0.4s ease, border-color 0.4s ease;
}

.event-card:focus-visible {
  outline: 2px solid var(--primary-accent, #3c763d);
  outline-offset: 4px;
}

.event-card:hover {
  transform: translateY(-6px) scale(1.015);
  box-shadow: 0 24px 30px rgba(0, 0, 0, 0.12),
    inset 0 1px 0 rgba(255, 255, 255, 1);
  border-color: rgba(255, 255, 255, 0.9);
  background: rgba(255, 255, 255, 0.95);
}

.event-card-main {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1.5rem;
}

.event-info {
  flex: 1 1 300px;
}

.event-name {
  font-size: 1.8rem;
  font-weight: 800;
  color: #1a202c;
  line-height: 1.25;
  margin-bottom: 0.5rem;
  letter-spacing: -0.02em;
}

.event-name a {
  text-decoration: none;
  background-image: linear-gradient(
    var(--primary-accent, #3c763d),
    var(--primary-accent, #3c763d)
  );
  background-size: 0% 2px;
  background-repeat: no-wrap;
  background-position: left bottom;
  padding-bottom: 2px;
  transition: background-size 0.3s cubic-bezier(0.25, 0.8, 0.25, 1),
    color 0.3s ease;
  color: inherit;
}

.event-card:hover .event-name a {
  background-size: 100% 2px;
  color: var(--primary-accent, #3c763d);
}

.event-period {
  font-size: 0.95rem;
  color: var(--text-muted, #718096);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.event-meta {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  flex: 0 0 auto;
}

.event-location {
  color: white;
  padding: 0.6rem 1.4rem;
  border-radius: 9999px;
  font-weight: 600;
  font-size: 1.15rem;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
}

.event-upcoming {
  text-align: right;
  min-width: 120px;
}

.event-upcoming-date {
  font-size: 1.35rem;
  font-weight: 800;
  color: #1a202c;
}

.event-upcoming-time {
  font-size: 0.95rem;
  color: var(--text-muted, #718096);
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 0.3rem;
  margin-top: 0.2rem;
}

.meta-icon {
  width: 18px;
  height: 18px;
  opacity: 0.85;
}

.event-location .meta-icon {
  width: 16px;
  height: 16px;
  opacity: 1;
}

/* Expanded Details */
.event-expanded {
  margin-top: 1.5rem;
  padding-top: 1.5rem;
  border-top: 1px solid rgba(0, 0, 0, 0.08);
}

.expanded-header {
  font-weight: 700;
  color: #4a5568;
  margin-bottom: 1rem;
  font-size: 1.15rem;
}

.expanded-dates {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1rem;
}

.expanded-date-item {
  display: flex;
  align-items: center;
  background: rgba(0, 0, 0, 0.03);
  padding: 0.85rem 1rem;
  border-radius: 8px;
  border-left: 4px solid var(--primary-accent, #3c763d);
  gap: 0.75rem;
  white-space: nowrap;
}

.date-day {
  font-weight: 700;
  color: var(--text-main, #2d3748);
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.date-day .meta-icon {
  width: 16px;
  height: 16px;
  color: var(--primary-accent, #3c763d);
}

.date-time {
  color: var(--text-muted, #718096);
  font-size: 0.95rem;
  font-weight: 500;
}

.date-room {
  font-size: 0.85rem;
  color: var(--text-main, #2d3748);
  font-weight: 600;
  margin-left: 0.75rem;
  background: var(--room-bg, rgba(60, 118, 61, 0.1));
  padding: 0.25rem 0.6rem;
  border-radius: 999px;
  display: inline-flex;
  align-items: center;
}

/* Themes */
.theme-default {
  --primary-accent: #3c763d;
  --theme-dark: #2f5c30;
  --room-bg: rgba(60, 118, 61, 0.1);
  --header-text: #ffffff;
}

.theme-eurac {
  background-color: #f29400 !important;
  --primary-accent: #f29400;
  --theme-dark: #cc7d00;
  --room-bg: rgba(242, 148, 0, 0.15);
  --header-text: #ffffff;
}

.theme-eurac .event-location {
  background-image: linear-gradient(
    135deg,
    #f29400 0%,
    #cc7d00 100%
  ) !important;
}

.theme-noi {
  background-color: #000000 !important;
  --primary-accent: #000000;
  --theme-dark: #111111;
  --room-bg: rgba(0, 0, 0, 0.1);
  --header-text: #ffffff;
}

.theme-noi .event-location {
  background-image: linear-gradient(
    135deg,
    #333333 0%,
    #000000 100%
  ) !important;
}

/* Skeleton Loading Shimmer */
.skeleton-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: default;
  background: rgba(255, 255, 255, 0.4);
}

.skeleton-card:hover {
  transform: none;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05),
    inset 0 1px 0 rgba(255, 255, 255, 0.8);
  border-color: rgba(255, 255, 255, 0.6);
  background: rgba(255, 255, 255, 0.4);
}

.skeleton-main {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  flex: 1 1 300px;
}

.skeleton-title,
.skeleton-subtitle,
.skeleton-pill,
.skeleton-date {
  background: #e2e8f0;
  background: linear-gradient(90deg, #e2e8f0 25%, #f7fafc 50%, #e2e8f0 75%);
  background-size: 400% 100%;
  animation: shimmer 1.5s infinite linear;
  border-radius: 8px;
}

.skeleton-title {
  height: 32px;
  width: 65%;
}

.skeleton-subtitle {
  height: 18px;
  width: 40%;
}

.skeleton-meta {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.skeleton-pill {
  height: 42px;
  width: 140px;
  border-radius: 9999px;
}

.skeleton-date {
  height: 30px;
  width: 95px;
}

@keyframes shimmer {
  0% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0 50%;
  }
}

/* Empty State */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 8rem 2rem;
  color: rgba(255, 255, 255, 0.95);
  text-align: center;
}

.empty-icon-wrapper {
  background: rgba(255, 255, 255, 0.15);
  padding: 1.5rem;
  border-radius: 50%;
  margin-bottom: 2rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.empty-icon {
  width: 64px;
  height: 64px;
  opacity: 1;
}

.empty-state h3 {
  font-size: 2.2rem;
  margin-bottom: 0.5rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  color: white;
}

.empty-state p {
  font-size: 1.15rem;
  opacity: 0.85;
}

/* List Stagger Animations */
.list-stagger-item {
  animation: slideFadeUp 0.6s cubic-bezier(0.16, 1, 0.3, 1) both;
}

@keyframes slideFadeUp {
  0% {
    opacity: 0;
    transform: translateY(30px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Expand Icon */
.expand-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #a0aec0;
  transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  margin-left: 0.5rem;
}

.is-expanded .expand-icon,
.expand-icon.is-rotated {
  transform: rotate(180deg);
}

.date-bullet {
  display: none;
}

.date-day {
  font-weight: 600;
  margin-right: auto;
  font-size: 1.05rem;
}

.date-time {
  font-family: monospace;
  background: rgba(0, 0, 0, 0.06);
  padding: 0.35rem 0.65rem;
  border-radius: 6px;
  font-size: 0.95em;
  font-weight: 600;
}

/* Transitions */
.expand-enter-active,
.expand-leave-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  max-height: 1000px;
  opacity: 1;
  overflow: hidden;
}
.expand-enter,
.expand-leave-to {
  max-height: 0;
  opacity: 0;
  padding-top: 0;
  padding-bottom: 0;
  margin-top: 0;
}

.events-footer {
  margin-top: 2rem;
  text-align: right;
  padding-right: 1rem;
}

.events-footer a {
  color: rgba(255, 255, 255, 0.85);
  font-size: 1rem;
  font-weight: 500;
  display: inline-flex;
  align-items: center;
  gap: 0.75rem;
  transition: color 0.2s;
}

.events-footer a:hover {
  color: white;
}

.events-footer img {
  height: 28px;
}

@media (max-width: 768px) {
  .events-header {
    flex-direction: column;
    align-items: flex-start;
    padding: 1.5rem;
    gap: 1rem;
  }
  .events-datetime {
    flex-direction: column;
    gap: 0.25rem;
  }
  .event-meta {
    width: 100%;
    justify-content: space-between;
    margin-top: 1rem;
  }
  .event-upcoming {
    text-align: right;
  }
}
</style>
