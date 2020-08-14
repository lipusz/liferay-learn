# Crafting XML Workflow Definitions

All Workflow definitions in Liferay DXP are written in XML format. This article documents introduces the basics of creating a workflow definition, such as the schema and required metadata.

```tip::
   Subscribers using DXP have the option of using a graphical designer to edit their workflows. They have the option to upload an XML file and then continue configuring their workflows.
```

## Existing Workflow Definitions

Only one workflow definition is installed by default: Single Approver. Several more are embedded in the source code of the Liferay DXP installation. These definitions provide good reference material for many of the workflow features and elements described in these articles.

* [Category Specific](../user-guide/workflow-designer-overview/workflow-processes/category-specific-definition.xml)
* [Legal Marketing](../user-guide/workflow-designer-overview/workflow-processes/legal-marketing-definition.xml)
* [Single Approver](../user-guide/workflow-designer-overview/workflow-processes/single-approver-definition.xml)
* [Single Approver Scripted Assignment](../user-guide/workflow-designer-overview/workflow-processes/single-approver-definition-scripted-assignment.xml)

## Schema

The XML structure of a workflow definition is defined in this XSD file: `liferay-workflow-definition-7_2_0.xsd`.

Declare the schema at the top of the workflow definition file:

```xml
<?xml version="1.0"?>
<workflow-definition
    xmlns="urn:liferay.com:liferay-workflow_7.2.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="urn:liferay.com:liferay-workflow_7.2.0
        http://www.liferay.com/dtd/liferay-workflow-definition_7_2_0.xsd">
```

To learn more, see this [XSD file](https://www.liferay.com/dtd/liferay-workflow-definition_7_2_0.xsd).

## Metadata

Give the definition a name, description, and version:

```xml
<name>Category Specific Approval</name>
<description>A single approver can approve a workflow content.</description>
<version>1</version>
```

All these tags are optional. If present the first time a definition is saved,the `<name>` tag serves as a unique identifier for the definition. If not specified (or added sometime after the first save), a random unique name is generated and used to identify the workflow.

## Additional Information

* [Managing Workflows](../user-guide/managing-workflows.md)
* [Introduction to the Workflow Designer](../user-guide/workflow-designer-overview.md)