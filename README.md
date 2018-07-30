# FullCalendarBundle

[![Packagist](https://img.shields.io/packagist/v/toiba/fullcalendar-bundle.svg)](https://packagist.org/packages/toiba/fullcalendar-bundle)
[![Build Status](https://travis-ci.org/toiba/FullCalendarBundle.svg)](https://travis-ci.org/toiba/FullCalendarBundle)

This bundle allow you to integrate [FullCalendar.js](http://fullcalendar.io/) library in your Symfony3 or Symfony4 project.

## Requirements
* FullCalendar.js v3.9.0
* Symfony 3.0/4.0
* PHP v5.6+
* PHPSpec

## Installation

1. [Download FullCalendarBundle using composer](#1-download-fullcalendarbundle-using-composer)
2. [Enable bundle](#2-enable-bundle)
3. [Add Routing](#3-define-routes)
4. [Create your listener](#4-create-your-listener)
5. [Add styles and scripts in your template](#5-add-styles-and-scripts-in-your-template)

### 1. Download FullCalendarBundle using composer

```sh
$ composer require toiba/fullcalendar-bundle
```

### 2. Enable bundle

```php
// app/AppKernel.php

public function registerBundles()
{
    return array(
        //...
        new Toiba\FullCalendarBundle\FullCalendarBundle(),
    );
}
```

### 3. Define routes

```yaml
# app/config/routing.yaml

toiba_fullcalendar:
    resource: "@FullCalendarBundle/Resources/config/routing.yaml"
```

### 4. Create your listener
You need to create your listener/subscriber class in order to load your events data in the calendar.

```yaml
# app/config/services.yaml
services:
    AppBundle\EventListener\FullCalendarListener:
        tags:
            - { name: 'kernel.event_listener', event: 'fullcalendar.set_data', method: loadData }
```

This listener is called when the event 'fullcalendar.set_data' is launched, for this reason you will need add this in your services.yaml.

```php
// src/AppBundle/EventListener/FullCalendarListener.php

<?php

namespace AppBundle\EventListener;

use Toiba\FullCalendarBundle\Entity\Event;
use Toiba\FullCalendarBundle\Event\CalendarEvent;

class FullCalendarListener
{
    /**
     * @param CalendarEvent $calendarEvent
     *
     * @return FullCalendarEvent[]
     */
    public function loadData(CalendarEvent $calendar)
    {
        $startDate = $calendar->getStart();
        $endDate = $calendar->getEnd();
        $filters = $calendar->getFilters(); // array of ajax sended data ex: 'foo' => 'bar'

        // You may want do a custom query to populate the calendar
        // b.beginAt is the start date in the booking entity

        $bookings = $this->em->getRepository(Booking::class)
            ->createQueryBuilder('b')
            ->andWhere('b.beginAt BETWEEN :startDate and :endDate')
            ->setParameter('startDate', $startDate->format('Y-m-d H:i:s'))
            ->setParameter('endDate', $endDate->format('Y-m-d H:i:s'))
            ->getQuery()->getResult();

        foreach($bookings as $booking) {

            // create an event
            $bookingEvent = new Event(
                $booking->getTitle(),
                $booking->getBeginAt(),
                $booking->getEndAt() // If end date is null or not defined, it create an all day event
            );

            /*
             * For more information see : Toiba\FullCalendarBundle\Entity\Event
             * and : https://fullcalendar.io/docs/event-object
             */
            // $bookingEvent->setBackgroundColor($booking['bgColor']);
            // $bookingEvent->setCustomField('borderColor', $booking['bgColor']);

            // finally, add the booking to the CalendarEvent for displaying on the calendar
            $calendar->addEvent($bookingEvent);
        }
    }
}
```

### 5. Add styles and scripts in your template

Add html template to display the calendar:

```twig
{% block body %}
    {% include '@FullCalendar/Calendar/calendar.html.twig' %}
{% endblock %}
```

Add styles:

```twig
{% block stylesheets %}
    <link rel="stylesheet" href="{{ asset('bundles/fullcalendar/css/fullcalendar/fullcalendar.min.css') }}" />
{% endblock %}
```

Add javascript:

```twig
{% block javascripts %}
    <script type="text/javascript" src="{{ asset('bundles/fullcalendar/js/fullcalendar/lib/jquery.min.js') }}"></script>
    <script type="text/javascript" src="{{ asset('bundles/fullcalendar/js/fullcalendar/lib/moment.min.js') }}"></script>
    <script type="text/javascript" src="{{ asset('bundles/fullcalendar/js/fullcalendar/fullcalendar.min.js') }}"></script>
{% endblock %}
```

# Basic functionalities

```js
$(function () {
    $('#calendar-holder').fullCalendar({
        header: {
            left: 'prev, next',
            center: 'title',
            right: 'month, basicWeek, basicDay,'
        },
        lazyFetching: true,
        timeFormat: {
            agenda: 'h:mmt',
            '': 'h:mmt'
        },
        eventSources: [
            {
                url: "{{ path('fullcalendar_load_events') }}",
                type: 'POST',
                data: {},
                error: function() {
                   alert('There was an error while fetching FullCalendar!')
                }
            }
        ]
    });
});
```

# Extending Basic functionalities

## Extending the Calendar Javascript
If you want to customize the FullCalendar javascript you can copy the fullcalendar.default-settings.js in YourBundle/Resources/public/js, and add your own logic:

```js
$(function () {
    $('#calendar-holder').fullCalendar({
        header: {
            left: 'prev, next, today',
            center: 'title',
            right: 'month, agendaWeek, agendaDay'
        },
        timezone: ('Europe/Paris'),
        businessHours: {
            start: '09:00',
            end: '18:00',
            dow: [1, 2, 3, 4, 5]
        },
        allDaySlot: false,
        defaultView: 'agendaWeek',
        lazyFetching: true,
        firstDay: 1,
        selectable: true,
        timeFormat: {
            agenda: 'h:mmt',
            '': 'h:mmt'
        },
        columnFormat:{
            month: 'ddd',
            week: 'ddd D/M',
            day: 'dddd'
        },
        editable: true,
        eventDurationEditable: true,
        eventSources: [
            {
                url: "{{ path('fullcalendar_load_events') }}",
                type: 'POST',
                data: {
                    filters: { 'foo': 'bar' }
                }
                error: function() {
                   alert('There was an error while fetching FullCalendar!')
                }
            }
        ]
    });
});
```

Contribute and feedback
-------------------------

Any feedback and contribution will be very appreciated.
