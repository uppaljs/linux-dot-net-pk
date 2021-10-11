---
title: 'Service Management Cheat Sheet :: CentOS 6 & 7'
author: uppal
type: post
date: 2014-11-30T06:54:15+00:00
url: /service-management-cheat-sheet-centos-6-7/
spacious_page_layout:
  - default_layout
categories:
  - DevOps
  - Howtos

---
Below is a conversion document for Centos 6 to 7 Service Management!

<table class="lt-4-cols gt-7-rows" summary="Comparison of the service Utility with systemctl ">
  <tr>
    <th>
      service
    </th>
    
    <th>
      systemctl
    </th>
    
    <th>
      Description
    </th>
  </tr>
  
  <tr>
    <td>
      <div class="para">
        <code class="command">service &lt;em class="replaceable">name&lt;/em> start</code>
      </div>
    </td>
    
    <td>
      <div class="para">
        <code class="command">systemctl start &lt;em class="replaceable">name&lt;/em>.service</code>
      </div>
    </td>
    
    <td>
      Starts a service.
    </td>
  </tr>
  
  <tr>
    <td>
      <div class="para">
        <code class="command">service &lt;em class="replaceable">name&lt;/em> stop</code>
      </div>
    </td>
    
    <td>
      <div class="para">
        <code class="command">systemctl stop &lt;em class="replaceable">name&lt;/em>.service</code>
      </div>
    </td>
    
    <td>
      Stops a service.
    </td>
  </tr>
  
  <tr>
    <td>
      <div class="para">
        <code class="command">service &lt;em class="replaceable">name&lt;/em> restart</code>
      </div>
    </td>
    
    <td>
      <div class="para">
        <code class="command">systemctl restart &lt;em class="replaceable">name&lt;/em>.service</code>
      </div>
    </td>
    
    <td>
      Restarts a service.
    </td>
  </tr>
  
  <tr>
    <td>
      <div class="para">
        <code class="command">service &lt;em class="replaceable">name&lt;/em> condrestart</code>
      </div>
    </td>
    
    <td>
      <div class="para">
        <code class="command">systemctl try-restart &lt;em class="replaceable">name&lt;/em>.service</code>
      </div>
    </td>
    
    <td>
      Restarts a service only if it is running.
    </td>
  </tr>
  
  <tr>
    <td>
      <div class="para">
        <code class="command">service &lt;em class="replaceable">name&lt;/em> reload</code>
      </div>
    </td>
    
    <td>
      <div class="para">
        <code class="command">systemctl reload &lt;em class="replaceable">name&lt;/em>.service</code>
      </div>
    </td>
    
    <td>
      Reloads configuration.
    </td>
  </tr>
  
  <tr>
    <td>
      <div class="para">
        <code class="command">service &lt;em class="replaceable">name&lt;/em> status</code>
      </div>
    </td>
    
    <td>
      <div class="para">
        <code class="command">systemctl status &lt;em class="replaceable">name&lt;/em>.service</code>
      </div>
      
      <div class="para">
        <code class="command">systemctl is-active &lt;em class="replaceable">name&lt;/em>.service</code>
      </div>
    </td>
    
    <td>
      Checks if a service is running.
    </td>
  </tr>
  
  <tr>
    <td>
      <div class="para">
        <code class="command">service --status-all</code>
      </div>
    </td>
    
    <td>
      <div class="para">
        <code class="command">systemctl list-units --type service --all</code>
      </div>
    </td>
    
    <td>
      Displays the status of all services.
    </td>
  </tr>
</table>

Here&#8217;s a conversion for **chkconfig**

<div class="table">
  <div class="table-contents">
    <table class="lt-4-cols lt-7-rows" summary="Comparison of the chkconfig Utility with systemctl">
      <tr>
        <th>
          chkconfig
        </th>
        
        <th>
          systemctl
        </th>
        
        <th>
          Description
        </th>
      </tr>
      
      <tr>
        <td>
          <div class="para">
            <code class="command">chkconfig &lt;em class="replaceable">name&lt;/em> on</code>
          </div>
        </td>
        
        <td>
          <div class="para">
            <code class="command">systemctl enable &lt;em class="replaceable">name&lt;/em>.service</code>
          </div>
        </td>
        
        <td>
          Enables a service.
        </td>
      </tr>
      
      <tr>
        <td>
          <div class="para">
            <code class="command">chkconfig &lt;em class="replaceable">name&lt;/em> off</code>
          </div>
        </td>
        
        <td>
          <div class="para">
            <code class="command">systemctl disable &lt;em class="replaceable">name&lt;/em>.service</code>
          </div>
        </td>
        
        <td>
          Disables a service.
        </td>
      </tr>
      
      <tr>
        <td>
          <div class="para">
            <code class="command">chkconfig --list &lt;em class="replaceable">name&lt;/em></code>
          </div>
        </td>
        
        <td>
          <div class="para">
            <code class="command">systemctl status &lt;em class="replaceable">name&lt;/em>.service</code>
          </div>
          
          <div class="para">
            <code class="command">systemctl is-enabled &lt;em class="replaceable">name&lt;/em>.service</code>
          </div>
        </td>
        
        <td>
          Checks if a service is enabled.
        </td>
      </tr>
      
      <tr>
        <td>
          <div class="para">
            <code class="command">chkconfig --list</code>
          </div>
        </td>
        
        <td>
          <div class="para">
            <code class="command">systemctl list-unit-files --type service</code>
          </div>
        </td>
        
        <td>
          Lists all services and checks if they are enabled.
        </td>
      </tr>
    </table>
  </div>
</div>

<div class="section">
</div>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->