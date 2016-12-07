<template>
    <div ref="calendar" class="full-calendar-container"></div>
</template>

<script>
    require('fullcalendar')

    export default {
        props: {
            events: {
                type: Array,
                default: []
            },
            eventSources: {
                type: Array,
                default: []
            },
            editable: {
                type: Boolean,
                default: true
            },
            selectable: {
                type: Boolean,
                default: true
            },
            selectHelper: {
                type: Boolean,
                default: false
            },
            allDaySlot: {
                type: Boolean,
                default: true
            },
            header: {
                type: Object,
                default: {
                    left:   'prev,next today',
                    center: 'title',
                    right:  'month,agendaWeek,agendaDay'
                }
            },
            defaultView: {
                type: String,
                default: 'agendaWeek'
            },
            sync: {
                type: Boolean,
                default: false
            }
        },

        mounted() {
            const cal = $(this.$refs.calendar);

            cal.fullCalendar({
                header: this.header,
                defaultView: this.defaultView,
                editable: this.editable,
                selectable: this.selectable,
                selectHelper: this.selectHelper,
                allDaySlot: this.allDaySlot,
                aspectRatio: 2,
                timeFormat: 'HH:mm',
                events: this.events,
                eventSources: this.eventSources,

                eventRender: (event, element) => {
                    if (this.sync) {
                        this.events = cal.fullCalendar('clientEvents')
                    }
                },

                eventDestroy: (event) => {
                    if (this.sync) {
                        this.events = cal.fullCalendar('clientEvents')
                    }
                },

                eventClick: (event) => {
                    this.$emit('event-selected', event)
                },

                eventDrop: (event) => {
                    this.$emit('event-drop', event)
                },

                eventResize: (event) => {
                    this.$emit('event-resize', event)
                },

                select: (start, end, jsEvent) =>  {
                    this.$emit('event-created', {
                        start,
                        end,
                        allDay: !start.hasTime() && !end.hasTime(),
                    })
                },
            })
        },

        watch: {
            events: {
                deep: true,
                handler(val) {
                    this.$emit('reload-events')
                },
            }
        },

        created() {
            this.$on('remove-event', (event) => {
                $(this.$refs.calendar).fullCalendar('removeEvents', event.id)
            })
            this.$on('rerender-events', (event) => {
                $(this.$refs.calendar).fullCalendar('rerenderEvents')
            })
            this.$on('refetch-events', (event) => {
                $(this.$refs.calendar).fullCalendar('refetchEvents')
            })
            this.$on('render-event', (event) => {
                $(this.$refs.calendar).fullCalendar('renderEvent', event)
            })
            this.$on('reload-events', () => {
                $(this.$refs.calendar).fullCalendar('removeEvents')
                $(this.$refs.calendar).fullCalendar('addEventSource', this.events)
            })
            this.$on('rebuild-sources', () => {
                $(this.$refs.calendar).fullCalendar('removeEvents')
                this.eventSources.map(event => {
                    $(this.$refs.calendar).fullCalendar('addEventSource', event)
                })
            })
        }
    }
</script>
