
# Angular Thermal Printer Library

A library for connecting Angular apps with thermal printers (type ESC/POS or Start models).

## Drivers

1. WebUSB API (No drivers needed. Works only with Chrome, Edge and Opera with direct USB connection).
Why Chrome, Edge and Opera only? The USB interface of the WebUSB API provides attributes and methods for finding and connecting USB devices from a web page, but it is an experimental technology (that works, indeed).

2. WebPRNT (http, Star printers only)
What is WebPRNT? The Star Company developed WebPRNT™ solution, that is designed to generate print data via a web browser to output directly to any Star printer fitted with a WebPRNT interface or using Star’s WebPRNT Browser Software. This includes Star thermal receipt, ticket & label printers as well as SP700 matrix kitchen printer. So, this remote printing technology will work only with Star printers.

## Developers important note

If you are getting this error when trying to get USB devices in your remote server (maybe production server):
`Cannot read property 'requestDevice' of undefined at Observable._subscribe (ng-thermal-print.js:1037)`
but all is running right in localhost, the official docs about **USB.requestDevice()** says: _This feature is available only in secure contexts (HTTPS)_
So you need to set your remote server to use HTTPS. It works in localhost because it is a "secure" place too.

## Print Language Drivers

1. ESC/POS
2. StarPRNT
3. Star WebPRNT

## Installation

Install library

`npm install ng-thermal-print`

NOTE: if when compiling project you get an error like this:
`ERROR in node_modules/ng-thermal-print/lib/drivers/UsbDriver.d.ts:16:30 - error TS2304: Cannot find name 'USBDevice'. requestUsb(): Observable;`
then add reference to w3c-web-usb by installing it with command: `npm install @types/w3c-web-usb --only=dev`

Import into your application

    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';
    import { ThermalPrintModule } from 'ng-thermal-print'; //add this line
    import { AppComponent } from './app.component';

    @NgModule({
       declarations: [
            AppComponent
        ],
        imports: [
            BrowserModule,
            ThermalPrintModule //add this line
        ],
        providers: [],
        bootstrap: [AppComponent]
    })
    export class AppModule { }

## Example Usage

app.component.ts

    import { PrintService, UsbDriver, WebPrintDriver } from 'ng-thermal-print';
    import { Component } from '@angular/core';
    import { PrintDriver } from 'ng-thermal-print/lib/drivers/PrintDriver';

    @Component({
        selector: 'app-root',
        templateUrl: './app.component.html',
        styleUrls: ['./app.component.css']
    })
    export class AppComponent {
        status: boolean = false;
        usbPrintDriver: UsbDriver;
        webPrintDriver: WebPrintDriver;
        ip: string = '';

        constructor(private printService: PrintService) {
            this.usbPrintDriver = new UsbDriver();
            this.printService.isConnected.subscribe(result => {
                this.status = result;
                if (result) {
                    console.log('Connected to printer!!!');
                } else {
                console.log('Not connected to printer.');
                }
            });
        }

        requestUsb() {
            this.usbPrintDriver.requestUsb().subscribe(result => {
                this.printService.setDriver(this.usbPrintDriver, 'ESC/POS');
            });
        }

        connectToWebPrint() {
            this.webPrintDriver = new WebPrintDriver(this.ip);
            this.printService.setDriver(this.webPrintDriver, 'WebPRNT');
        }

        print(driver: PrintDriver) {
            this.printService.init()
                .setBold(true)
                .writeLine('Hello World!')
                .setBold(false)
                .feed(4)
                .cut('full')
                .flush();
        }
    }

app.component.html

    <strong>Status: {{status}}</strong>
    <div>
        <input [(ngModel)]="ip" type="text" name="ip" placeholder="IP of printer with WebPRNT">
        <button (click)="connectToWebPrint()">Connect to WebPRNT</button>
    </div>

    <div>
        <button (click)="requestUsb()">Connect to USB</button>
    </div>

    <div>
        <button (click)="print()" [disabled]="status === false"> Print!</button>
    </div>
