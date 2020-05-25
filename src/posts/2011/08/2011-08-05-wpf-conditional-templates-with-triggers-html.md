---
title: "WPF Conditional Templates with Triggers"
date: "2011-08-05"
---

WPF has a lot of power and flexibility which is great but that also means there are several ways to do the same thing. That helped trip me up for a little bit this afternoon when I was trying to remember details on triggers.  
  

I had a GridView bound to a collection of DatabaseChange POCO objects with IsAttachment and Schema properties. I simply wanted the Schema cell to be editable when IsAttachment was True and read-only otherwise. I went through a couple different variations but settled on the below.  
  

The view's resources contains a DataTemplate for each of the states, readonly and editable. Another DataTemplate defines a ContentPresenter and a trigger that swaps out the ContentTemplate on the presenter if the DataTrigger binding condition is satisified.  

\[xml\] <UserControl.Resources> <DataTemplate x:Key="SchemaColumnEditable"> <ComboBox IsEditable="True" ItemsSource="{Binding DataContext.Schemas, RelativeSource={RelativeSource Mode=FindAncestor, AncestorType=DockPanel}}" Text="{Binding Schema, Mode=TwoWay}" /> </DataTemplate>

<DataTemplate x:Key="SchemaColumnReadOnly"> <TextBlock Text="{Binding Schema, Mode=OneWay}"/> </DataTemplate> <DataTemplate x:Key="schemaDetails"> <ContentPresenter x:Name="schemaContentPresenter" ContentTemplate="{StaticResource SchemaColumnReadOnly}" Content="{TemplateBinding Content}" />

<DataTemplate.Triggers> <DataTrigger Binding="{Binding IsAttachment}" Value="True"> <Setter TargetName="schemaContentPresenter" Property="ContentTemplate" Value="{StaticResource SchemaColumnEditable}" /> </DataTrigger> </DataTemplate.Triggers> </DataTemplate> </UserControl.Resources> \[/xml\]  

Use of the template in the GridView:  

\[xml\] <!-- ListView attributes and other GridView columns removed for brevity --> <ListView> <ListView.View> <GridView> <GridViewColumn Header="Schema" Width="90" CellTemplate="{StaticResource schemaDetails}" /> </GridView> </ListView.View> </ListView> \[/xml\]  

Simple enough. It's also worth nothing to not forget to remove the DisplayMemberBinding property on the GridViewColumn when changing it to use a cell template as otherwise the template will not take effect.  
  

This is a part of a "Database Packager" feature in a larger set of build management apps I have been working on since inheriting dreaded release engineer responsibilities from a project manager who left the company and has been sorely missed. Hopefully there will be more here on the release management front later if I can make the time...
