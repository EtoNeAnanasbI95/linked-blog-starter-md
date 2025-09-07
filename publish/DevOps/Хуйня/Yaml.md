# Anchor/alias
* Короче в этой хуйне есть такая тема как якоря и ссылки, то есть можно сделать вот такую парашу
```yaml
service:
	zeliboba: &zeliboba-anchor
		ports: |
			- "80:80"
			- "443:443"
		containter_name: "zeliboba"
		
		
	zeliboba-2:
		volumes: |
			- ./config.json:/etc/config/config.json
		<<: *zeliboba-anchor
```
* И в итоге мы получим типа вот эту хуйню:
```yaml
service:
	zeliboba:
		network: zeliboba
		ports: |
			- "80:80"
			- "443:443"
		
		
	zeliboba-2:
		volumes: |
			- ./config.json:/etc/config/config.json
		network: zeliboba
		ports: |
			- "80:80"
			- "443:443"
```