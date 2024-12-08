## nz-tab页签之间互相跳转，利用父子组件传递参数

### 背景

- 实现功能：有两个`nz-tab`标签 tab1， tab2，假如 tab1 页签上有个按钮，点击后， 需要跳转到tab2， 并将tab1页签的某些值传递给tab2，同时调用tab2的查询方法，让tab2页面重新查询
- 实现思路：两个`nz-ta`控制使用`nzSelectedIndex`控制页签的展示，接下来就是子组件先向父组件发送切换页签的事件（使用event.emit），父组件接受到参数后，调用另一个子组件的查询函数（使用 @ViewChild 装饰器来调用子组件的方法称为` 依赖注入` 或 `子组件方法调用`）

### 代码实现

#### 父组件

- html代码

  ```html
   <nz-tabset [nzAnimated]="false" [(nzSelectedIndex)]="selectedIndex">
      <nz-tab nzTitle="基金主页">
        <app-security-homepage #securityRef (event)="navigateToTab2($event)"></app-security-homepage>
      </nz-tab>
      <nz-tab nzTitle="参与者主页">
        <app-participant-homepage #participantRef (event)="navigateToTab1($event)"></app-participant-homepage>
      </nz-tab>
    </nz-tabset>
    <router-outlet></router-outlet>
  ```

  

- ts代码

  ```ts
  // 在父组件中导入 ViewChild 装饰器
  import { Component, OnInit, Output, Input, ViewChild } from '@angular/core';
  import { SecurityHomepageComponent } from './security-homepage/security-homepage.component';
  import { ParticipantHomepageComponent } from './participant-homepage/participant-homepage.component';
  @Component({
    selector: 'app-fund-homepage',
    templateUrl: './fund-homepage.component.html',
    styleUrls: ['./fund-homepage.component.less']
  })
  export class FundHomepageComponent implements OnInit {
    // @ViewChild 装饰器声明一个变量来引用子组件
    @ViewChild('securityRef', { static: false }) securityRef!: SecurityHomepageComponent;
    @ViewChild('participantRef', { static: false }) participantRef!: ParticipantHomepageComponent;
  
    constructor() {}
  
    selectedIndex = 0;
  
    navigateToTab1(param: any) {
      this.selectedIndex = 0;
      if (this.securityRef) {
        this.securityRef.handleJumpQueryEvent(param);
      }
    }
  
    navigateToTab2(param: any) {
      this.selectedIndex = 1;
      if (this.participantRef) {
          // 在父组件的适当位置调用子组件的方法
        this.participantRef.handleJumpQueryEvent(param);
      }
    }
  
    ngOnInit(): void {}
  }
  ```

  

#### 负责点击的子组件tab1

- html代码

  ```html
   <button nz-button nzType="primary" (click)="jump()">
        <i nz-icon nzType="search"></i>查询
      </button>
  ```

- ts代码

  ```ts
  import { FormGroup } from '@angular/forms';
  // 在子组件中导入 EventEmitter 装饰器
  import { AfterViewInit, Component, EventEmitter, OnInit, Output, Input, ViewChild } from '@angular/core';
  import { BehaviorSubject } from 'rxjs';
  import { finalize } from 'rxjs/operators';
  
  import { TableEvent } from 'ng-invest-antd/public-table';
  import { NzMessageService } from 'ng-zorro-antd/message';
  import { NzModalService } from 'ng-zorro-antd/modal';
  import { Axis, Series } from '@/components/public-components/public-chart/chart';
  
  import { BirPublicTableComponent } from '@/components/public-components/public-table/public-table.component';
  import { TableService } from '@/components/public-components/public-table/table.service';
  import { UtilService } from '@/components/public-components/util.service';
  import { FormService } from '@/core/services/common/form.service';
  import { HttpConfigService } from '@/core/services/common/http-config.service';
  import { HttpService } from '@/core/services/common/http.service';
  import { JsonErrorResponse } from '@/models/common/json-error-response.model';
  import _ from 'lodash';
  
  @Component({
    selector: 'app-security-homepage',
    templateUrl: './security-homepage.component.html',
    styleUrls: ['./security-homepage.component.less']
  })
  export class SecurityHomepageComponent implements OnInit {
    @Output() event = new EventEmitter<any>();
    
    constructor() {}
  
    ngOnInit(): void { }
  
    ngAfterViewInit(): void {}
  
    jump() {
        // 触发事件
       this.event.emit({
          begindate: '20240101',
          enddate:  '20240201',
          stklongname: ''
        });
    }
  }
  
  ```

#### 跳转后的子组件tab2

- ts代码

```ts
import { FormGroup } from '@angular/forms';

import { AfterViewInit, Component, EventEmitter, OnInit, Output, Input, ViewChild } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import { finalize } from 'rxjs/operators';

import { TableEvent } from 'ng-invest-antd/public-table';
import { NzMessageService } from 'ng-zorro-antd/message';
import { NzModalService } from 'ng-zorro-antd/modal';
import { Axis, Series } from '@/components/public-components/public-chart/chart';

import { BirPublicTableComponent } from '@/components/public-components/public-table/public-table.component';
import { TableService } from '@/components/public-components/public-table/table.service';
import { UtilService } from '@/components/public-components/util.service';
import { FormService } from '@/core/services/common/form.service';
import { HttpConfigService } from '@/core/services/common/http-config.service';
import { HttpService } from '@/core/services/common/http.service';
import { JsonErrorResponse } from '@/models/common/json-error-response.model';
import _ from 'lodash';


@Component({
  selector: 'app-participant-homepage',
  templateUrl: './participant-homepage.component.html',
  styleUrls: ['./participant-homepage.component.less']
})
export class ParticipantHomepageComponent implements OnInit {
  constructor() {}

  ngOnInit(): void { }

  ngAfterViewInit(): void {}
  query() {}
  
  // 在子组件中定义父组件需要调用的方法
  handleJumpQueryEvent(event: any) {
    console.log('跳转参数', JSON.stringify(event));

    this.formService.setValueByKey(this.form, this.formGroup, 'busidate', [
      new Date(event.begindate.slice(0, 4) + '-' + event.begindate.slice(4, 6) + '-' + event.begindate.slice(6, 8)),
      new Date(event.enddate.slice(0, 4) + '-' + event.enddate.slice(4, 6) + '-' + event.enddate.slice(6, 8))
    ]);
    this.formService.setValueByKey(this.form, this.formGroup, 'stklongname', [event.stklongname]);
    this.query();
  }
}

```
