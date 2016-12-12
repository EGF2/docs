# System ES Indexes

<style>
    .last {
    border-bottom: 1px solid #ccc;
   }
</style>

<table>
	<thead>
		<tr>
			<th>Index Name / subsystem</th>
			<th>Objects</th>
			<th>Fields</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td ROWSPAN=3>File / <b>file</b></td>
			<td>File</td>
			<td>field: id</td>
		</tr>
		<tr>
			<td></td>
			<td>filter: standalone</td>
		</tr>
		<tr>
			<td></td>
			<td>filter: created_at</td>
		</tr>
		<tr>
			<td>Schedule / <b>scheduler</b></td>
			<td>Schedule</td>
			<td>field: id</td>
		</tr>
		<tr>
        	<td ROWSPAN=2 class='last'>User / <b>auth</b></td>
        	<td>User</td>
        	<td>field: email</td>
        </tr>
        <tr>
        	<td></td>
        	<td>field: id</td>
        </tr>
	</tbody>
</table>
